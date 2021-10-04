---
title: Using Terraform to create Failover records in Route53
header:
  overlay_image: "/content/images/2018/11/nasa-53884-unsplash.jpg"
---
I was recently tasked with creating some DNS records in AWS's Route53 service using Terraform. These records were to use a Failover Routing policy so that traffic would be routed to a secondary site if the first was unavailable. While working on this I found the Terraform documentation around configuring failover in Route53 was a bit brief, so am documenting my code via this blog post.

> Note: This isn't necessarily best practice, or even correct practice, i'm still new to Terraform. Use with caution.

```Terraform
variable "primary_example_com" {}
variable "secndry_example_com" {}

resource "aws_route53_zone" "example_com" {
    name = "example.com"
}
  
# example.com
  
resource "aws_route53_record" "example_com_primary" {
    zone_id = "${aws_route53_zone.example_com.zone_id}"
    name    = "${aws_route53_zone.example_com.name}"
    type    = "A"
    ttl     = "60"
    records = ["${var.primary_example_com}"]
  
    failover_routing_policy {
        type = "PRIMARY"
    }
  
    set_identifier  = "example_com_primary"
    health_check_id = "${aws_route53_health_check.example_com_primary.id}"
}
  
resource "aws_route53_health_check" "example_com_primary" {
    ip_address        = "${var.primary_example_com}"
    port              = "80"
    type              = "HTTP"
    resource_path     = "/"
    failure_threshold = "3"
    request_interval  = "30"
}
  
resource "aws_route53_record" "example_com_secondary" {
    zone_id = "${aws_route53_zone.example_com.zone_id}"
    name    = "${aws_route53_zone.example_com.name}"
    type    = "A"
    ttl     = "60"
    records = ["${var.secndry_example_com}"]
  
    failover_routing_policy {
        type = "SECONDARY"
    }
  
    set_identifier  = "example_com_secondary"
    health_check_id = "${aws_route53_health_check.example_com_secondary.id}"
}
  
resource "aws_route53_health_check" "example_com_secondary" {
    ip_address        = "${var.secndry_example_com}"
    port              = "80"
    type              = "HTTP"
    resource_path     = "/"
    failure_threshold = "3"
    request_interval  = "30"
}
```