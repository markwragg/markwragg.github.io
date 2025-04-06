---
title: Configuring a managed identity as an Azure SQL user via an Azure DevOps pipeline
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/azure-sql.jpg"
  teaser: "/content/images/2025/azure-sql.jpg"
date: "2025-04-06 11:00:00"
tags:
  - azure
  - sql
  - azuredevops
  - powershell
---

I was recently tasked with enabling connectivity to an Azure SQL database via the managed identity of a Logic App and it was surprisingly complicated to setup. In addition we wanted this configuration to be enabled through our automation pipelines, which added a further complexity. This blog post details the steps we had to take.

Azure provides managed identities as a secure and convenient way to manage access to resources. By configuring a managed identity you no longer need to store secrets, keys or credentials. For SQL server, the traditional way to access a database would be via a connection string that would include a username and password. However managed identities are password-less, so eliminate the need to provide a password directly in the connection string.

The complexity of this task is due to the following:

## 1. To configure a managed identity as a user within Azure SQL the user must be configured by another Entra ID account

At first this might seem like a [chicken and egg scenario](https://en.wikipedia.org/wiki/Chicken_or_the_egg), because how can you configure an Entra ID account as a user without already having an Entra ID account as a user? The answer is to configure an Entra ID account as the SQL Server AD Administrator. If you attempt to connect using local SQL credentials (even those with system administrator permissions) and attempt to create a SQL login for an Entra ID user you'll get an error.

Because we wanted to be able to configure the Logic App managed identity with permissions to SQL via the pipeline, I made the service principal of the pipeline the Azure AD Administrator. I did this via PowerShell as follows (this script ran in the pipeline that deploys Azure SQL, which has new input variables to specify the name `$sqlAdAdminName` and object ID `$sqlAdAdminObjectId` of the pipeline service principal):

```powershell
# Get existing Sql Server configuration
$sqlServer = az sql server show --name $sqlServerName --resource-group $resourceGroup --output json | ConvertFrom-Json

# Check if AD admin already exists
$existingAdAdministrator = $sqlServer.administrators | Where-Object { $_.administratorType -eq 'ActiveDirectory' }

# Configure AD admin if not already set to the required SQL Admin name
if ($existingAdAdministrator.login -ne $sqlAdAdminName) {

    Write-Host "Setting SQL AD Admin.."
    az sql server ad-admin create --resource-group $resourceGroup --server $sqlServerName --display-name $sqlAdAdminName --object-id $sqlAdAdminObjectId

    if ($? -eq $false) {
        Write-Error "Error setting SQL AD Admin $sqlAdAdminName with objectId $sqlAdAdminObjectId."
    }
}
else {
    Write-Host "SQL AD Admin already set to $sqlAdAdminName."
}
```

If you didn't want to use the pipeline's service principal as the AD administrator, you could configure another Entra ID user or group and then provide the credentials for that user during your pipeline execution. Obviously using the pipeline's service principal negates the need to store any credentials, but it does mean anything executed via the pipeline's service principal in the future has AD Administrator access to the SQL Server, so would have to be carefully secured. It's up to you to decide whether this is appropriate for your specific database/scenario.

## 2. The SQL Server itself must be able to validate the Entra ID user you are configuring

You might think (as I did) that the Entra ID user you're now connecting to SQL Server with to execute your commands to create the new Entra ID login would have enough access by itself to complete the task. But actually, when you add a directory user to SQL Server, it seems SQL itself has to do a bit of validation against the directory. To do this, the Azure SQL Server needs either a System Assigned identity, or User Assigned identity configured with specific permissions to read the Entra ID directory.

You can enable a System Assigned identity via PowerShell as follows:

```powershell
# Check if existing System Assigned Identity is set
$existingSAIdentity = $sqlServer.identity | Where-Object { $_.type -eq 'SystemAssigned' }

# Configure System Assigned Identity if not already set
if (-not $existingSAIdentity) {

    Write-Host "Setting System Assigned Identity.."
    az sql server update --resource-group $resourceGroup --name $sqlServerName --assign_identity --identity-type SystemAssigned
}
else {
    Write-Host "System Assigned Identity already set."
}
```

Once you've configured a System Assigned identity you need to [grant it the Directory Readers role](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-directory-readers-role?view=azuresql). This has to be done by an Entra ID account with Privileged Role Administrator or higher permissions. We did this step manually, but it could be automated. Alternatively if you use a user-assigned managed identity, you can configure specific Microsoft Graph permissions for it via PowerShell as described in [this Microsoft article](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?view=azuresql).

## 3. Execute your SQL scripts to configure the managed identity permissions and roles

Having completed the above, we could now configure the Logic App managed identity as a user in the SQL server as follows (this script runs in another pipeline where the specific logic app is deployed that we wanted to grant access):

> Because we configured the pipeline user as a AD Admin, the below uses Az CLI to get the current users access token to authenticate to SQL. If you configured another user as SQL Server AD Admin you'll need to authenticate to SQL with those credentials.

```powershell
# Get the access token of the current pipeline user session
$accessToken = az account get-access-token --resource https://database.windows.net/ --query accessToken --output tsv

$invokeSqlcmd = @{
    ServerInstance  = "${sqlServerName}.database.windows.net"
    AccessToken     = $accessToken
    OutputSqlErrors = $true
}

# Specify the name of the Logic App resource, which has system assigned identity enabled
$laName = "your-logic-app-resource-name"

# Check if the DB user is already configured
try {
    $dbuserAlreadyExists = Invoke-Sqlcmd @invokeSqlcmd -Query "SELECT name FROM sys.sysusers WHERE name='$laName'" -Database $dbName -ErrorAction Stop
}
catch {
    Write-Error $_
    throw "Failed to check for existing login for id $laName in database $dbname."
}

if (-not $dbuserAlreadyExists) {

    Write-Verbose "Creating user $laName in database $dbName from external provider.."
    Invoke-Sqlcmd @invokeSqlcmd -Database $dbName -Query "CREATE USER [$laName] FROM EXTERNAL PROVIDER"

    Write-Verbose "Adding role db_datareader to user $laName.."
    Invoke-Sqlcmd @invokeSqlcmd -Database $dbName -Query "ALTER ROLE [db_datareader] ADD MEMBER [$laName];"
}
else {
    Write-Verbose "User $laName already exists."
}
```

For more information on the above (and to ensure you're following the latest Microsoft guidance) check out the official documentation for [configuring Microsoft Entra authentication with Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure).

The page on [Microsoft Entra service principals with Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal?view=azuresql) was also useful.