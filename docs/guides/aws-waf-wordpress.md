# AWS WAF for WordPress

## Architecture

```
Cloudflare (optional) → CloudFront → AWS WAF → ALB → WordPress (EC2/ECS)
                           or
Cloudflare (optional) → AWS WAF → ALB → WordPress (EC2/ECS)
```

## Prerequisites

- AWS account with permissions for WAF, CloudFront, and ALB
- WordPress running behind an ALB (Application Load Balancer)
- (Optional) CloudFront distribution in front of ALB

## Step 1: Create WAF Web ACL

Via AWS CLI:

```bash
# Create Web ACL
aws wafv2 create-web-acl \
  --name wordpress-waf \
  --scope REGIONAL \
  --default-action Allow={} \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=wordpress-waf

# Save the ARN and ID from the output
```

Via Terraform:

```hcl title="waf.tf"
resource "aws_wafv2_web_acl" "wordpress" {
  name        = "wordpress-waf"
  description = "WAF for WordPress site"
  scope       = "REGIONAL"

  default_action {
    allow {}
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "wordpress-waf"
    sampled_requests_enabled   = true
  }
}
```

## Step 2: Attach to ALB

```bash
# Get ALB ARN
aws elbv2 describe-load-balancers --names wordpress-alb --query 'LoadBalancers[0].LoadBalancerArn'

# Associate WAF with ALB
aws wafv2 associate-web-acl \
  --web-acl-arn <web-acl-arn> \
  --resource-arn <alb-arn>
```

Terraform:

```hcl
resource "aws_wafv2_web_acl_association" "wordpress" {
  resource_arn = aws_lb.wordpress.arn
  web_acl_arn  = aws_wafv2_web_acl.wordpress.arn
}
```

## Step 3: Managed Rule Groups

```bash
# Add AWS managed rules
aws wafv2 update-web-acl \
  --name wordpress-waf \
  --scope REGIONAL \
  --rules '[
    {
      "Name": "AWS-AWSManagedRulesCommonRuleSet",
      "Priority": 0,
      "Statement": { "ManagedRuleGroupStatement": { "VendorName": "AWS", "Name": "AWSManagedRulesCommonRuleSet" } },
      "OverrideAction": { "None": {} },
      "VisibilityConfig": { "SampledRequestsEnabled": true, "CloudWatchMetricsEnabled": true, "MetricName": "AWS-AWSManagedRulesCommonRuleSet" }
    },
    {
      "Name": "AWS-AWSManagedRulesSQLiRuleSet",
      "Priority": 1,
      "Statement": { "ManagedRuleGroupStatement": { "VendorName": "AWS", "Name": "AWSManagedRulesSQLiRuleSet" } },
      "OverrideAction": { "None": {} },
      "VisibilityConfig": { "SampledRequestsEnabled": true, "CloudWatchMetricsEnabled": true, "MetricName": "AWS-AWSManagedRulesSQLiRuleSet" }
    },
    {
      "Name": "AWS-AWSManagedRulesPHPRuleSet",
      "Priority": 2,
      "Statement": { "ManagedRuleGroupStatement": { "VendorName": "AWS", "Name": "AWSManagedRulesPHPRuleSet" } },
      "OverrideAction": { "None": {} },
      "VisibilityConfig": { "SampledRequestsEnabled": true, "CloudWatchMetricsEnabled": true, "MetricName": "AWS-AWSManagedRulesPHPRuleSet" }
    },
    {
      "Name": "AWS-AWSManagedRulesWordPressRuleSet",
      "Priority": 3,
      "Statement": { "ManagedRuleGroupStatement": { "VendorName": "AWS", "Name": "AWSManagedRulesWordPressRuleSet" } },
      "OverrideAction": { "None": {} },
      "VisibilityConfig": { "SampledRequestsEnabled": true, "CloudWatchMetricsEnabled": true, "MetricName": "AWS-AWSManagedRulesWordPressRuleSet" }
    }
  ]'
```

### Terraform Managed Rules

```hcl
resource "aws_wafv2_web_acl" "wordpress" {
  # ... (from above)

  rule {
    name     = "AWS-AWSManagedRulesCommonRuleSet"
    priority = 0

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesCommonRuleSet"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesCommonRuleSet"
      sampled_requests_enabled   = true
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesSQLiRuleSet"
    priority = 1

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesSQLiRuleSet"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesSQLiRuleSet"
      sampled_requests_enabled   = true
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesPHPRuleSet"
    priority = 2

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesPHPRuleSet"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesPHPRuleSet"
      sampled_requests_enabled   = true
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesWordPressRuleSet"
    priority = 3

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesWordPressRuleSet"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesWordPressRuleSet"
      sampled_requests_enabled   = true
    }
  }
}
```

### Managed Rule Groups Reference

| Rule Group | Protection |
|------------|------------|
| `AWSManagedRulesCommonRuleSet` | Generic attacks, bad inputs, XSS |
| `AWSManagedRulesSQLiRuleSet` | SQL injection |
| `AWSManagedRulesPHPRuleSet` | PHP-specific attacks |
| `AWSManagedRulesWordPressRuleSet` | WordPress-specific attacks |
| `AWSManagedRulesKnownBadInputsRuleSet` | Known bad patterns |
| `AWSManagedRulesAmazonIpReputationList` | Known malicious IPs |
| `AWSManagedRulesAnonymousIpList` | Anonymous IPs (VPN, Tor) |
| `AWSManagedRulesBotControlRuleSet` | Bot detection |

## Step 4: Custom Rules

### Rate Limiting

```bash
aws wafv2 update-web-acl \
  --name wordpress-waf \
  --scope REGIONAL \
  --rules '[
    ...
    {
      "Name": "wp-login-rate-limit",
      "Priority": 10,
      "Action": { "Block": {} },
      "Statement": {
        "RateBasedStatement": {
          "Limit": 100,
          "AggregateKeyType": "IP"
        }
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "wp-login-rate-limit"
      }
    }
  ]'
```

Terraform:

```hcl
  rule {
    name     = "wp-login-rate-limit"
    priority = 10

    action {
      block {}
    }

    statement {
      rate_based_statement {
        limit              = 100
        aggregate_key_type = "IP"

        scope_down_statement {
          regex_pattern_set_reference_statement {
            arn = aws_wafv2_regex_pattern_set.wp_paths.arn
            field_to_match {
              uri_path {}
            }
            text_transformation {
              priority = 0
              type     = "NONE"
            }
          }
        }
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "wp-login-rate-limit"
      sampled_requests_enabled   = true
    }
  }
}

resource "aws_wafv2_regex_pattern_set" "wp_paths" {
  name        = "wp-sensitive-paths"
  description = "WordPress sensitive paths"
  scope       = "REGIONAL"

  regular_expression {
    regex = "/wp-login\\.php"
  }
  regular_expression {
    regex = "/wp-admin"
  }
  regular_expression {
    regex = "/xmlrpc\\.php"
  }
}
```

### Block XML-RPC

```hcl
  rule {
    name     = "block-xmlrpc"
    priority = 11

    action {
      block {}
    }

    statement {
      byte_match_statement {
        positional_constraint = "EXACTLY"
        search_string         = "/xmlrpc.php"

        field_to_match {
          uri_path {}
        }

        text_transformation {
          priority = 0
          type     = "NONE"
        }
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "block-xmlrpc"
      sampled_requests_enabled   = true
    }
  }
```

### Allow Cloudflare IPs

If using Cloudflare in front of AWS, add this rule at high priority:

```hcl
  rule {
    name     = "allow-cloudflare"
    priority = 0

    action {
      allow {}
    }

    statement {
      ip_set_reference_statement {
        arn = aws_wafv2_ip_set.cloudflare.arn
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "allow-cloudflare"
      sampled_requests_enabled   = true
    }
  }
```

```hcl
data "http" "cloudflare_ips" {
  url = "https://www.cloudflare.com/ips-v4"
}

resource "aws_wafv2_ip_set" "cloudflare" {
  name               = "cloudflare-ips"
  description        = "Cloudflare IP ranges"
  scope              = "REGIONAL"
  ip_address_version = "IPV4"
  addresses          = split("\n", chomp(data.http.cloudflare_ips.response_body))
}
```

## Step 5: IP Blocklist/Allowlist

### Block Known Bad IPs

```hcl
resource "aws_wafv2_ip_set" "blocked_ips" {
  name               = "blocked-ips"
  description        = "Manually blocked IPs"
  scope              = "REGIONAL"
  ip_address_version = "IPV4"
  addresses          = ["1.2.3.4/32", "5.6.7.8/32"]
}

  rule {
    name     = "block-bad-ips"
    priority = 20

    action {
      block {}
    }

    statement {
      ip_set_reference_statement {
        arn = aws_wafv2_ip_set.blocked_ips.arn
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "block-bad-ips"
      sampled_requests_enabled   = true
    }
  }
```

### Allow Office IPs

```hcl
resource "aws_wafv2_ip_set" "office_ips" {
  name               = "office-ips"
  description        = "Office IP ranges"
  scope              = "REGIONAL"
  ip_address_version = "IPV4"
  addresses          = ["203.0.113.0/24"]
}

  rule {
    name     = "allow-office-ips"
    priority = 1

    action {
      allow {}
    }

    statement {
      ip_set_reference_statement {
        arn = aws_wafv2_ip_set.office_ips.arn
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "allow-office-ips"
      sampled_requests_enabled   = true
    }
  }
```

## Step 6: Logging & Monitoring

### Enable Logging

```bash
# Create CloudWatch log group
aws logs create-log-group --log-group-name /aws/waf/wordpress-waf

# Enable WAF logging
aws wafv2 put-logging-configuration \
  --logging-configuration ResourceArn=<web-acl-arn>,LogDestinationConfigs=["arn:aws:logs:<region>:<account>:log-group:/aws/waf/wordpress-waf:*"]
```

### CloudWatch Dashboard

```hcl
resource "aws_cloudwatch_dashboard" "waf" {
  dashboard_name = "waf-wordpress"

  dashboard_body = jsonencode({
    widgets = [
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/WAFV2", "BlockedRequests", "WebACL", "wordpress-waf", "Rule", "All"],
            ["AWS/WAFV2", "AllowedRequests", "WebACL", "wordpress-waf", "Rule", "All"]
          ]
          period = 300
          stat   = "Sum"
          region = "us-east-1"
          title  = "WAF Requests"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/WAFV2", "BlockedRequests", "WebACL", "wordpress-waf", "Rule", "AWS-AWSManagedRulesSQLiRuleSet"],
            ["AWS/WAFV2", "BlockedRequests", "WebACL", "wordpress-waf", "Rule", "AWS-AWSManagedRulesWordPressRuleSet"]
          ]
          period = 300
          stat   = "Sum"
          region = "us-east-1"
          title  = "Blocked by Rule Group"
        }
      }
    ]
  })
}
```

### Alert on High Block Rate

```hcl
resource "aws_cloudwatch_metric_alarm" "waf_high_block" {
  alarm_name          = "waf-high-block-rate"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "BlockedRequests"
  namespace           = "AWS/WAFV2"
  period              = "300"
  statistic           = "Sum"
  threshold           = "1000"
  alarm_description   = "WAF blocked more than 1000 requests in 5 minutes"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    WebACL = "wordpress-waf"
    Rule   = "All"
  }
}
```

## Step 7: Test WAF Rules

```bash
# Test SQL injection (should 403)
curl -X POST https://example.com/wp-login.php \
  -d "user=admin&pwd=1' OR '1'='1"

# Test XSS (should 403)
curl "https://example.com/?q=<script>alert(1)</script>"

# Test rate limiting
for i in $(seq 1 200); do curl -s -o /dev/null -w "%{http_code}\n" https://example.com/wp-login.php; done

# Check block in WAF logs
aws wafv2 get-web-acl --name wordpress-waf --scope REGIONAL
```

## IP Sets Management

```bash
# Add IP to blocklist
aws wafv2 update-ip-set \
  --name blocked-ips \
  --scope REGIONAL \
  --id <ip-set-id> \
  --addresses "1.2.3.4/32" "5.6.7.8/32" \
  --lock-token <lock-token>
```

## Rule Priority

Rules are evaluated in priority order (lower number = higher priority):

| Priority | Rule | Action |
|----------|------|--------|
| 0 | Allow Cloudflare IPs | Allow |
| 1 | Allow Office IPs | Allow |
| 2 | AWS Common Rule Set | Block |
| 3 | AWS SQLi Rule Set | Block |
| 4 | AWS PHP Rule Set | Block |
| 5 | AWS WordPress Rule Set | Block |
| 10 | Rate Limit wp-login | Block |
| 11 | Block XML-RPC | Block |
| 20 | Block Known Bad IPs | Block |

## Full Terraform Example

```hcl
resource "aws_wafv2_web_acl" "wordpress" {
  name        = "wordpress-waf"
  description = "WAF for WordPress on ALB"
  scope       = "REGIONAL"

  default_action { allow {} }

  # Allow Cloudflare
  rule {
    name     = "allow-cloudflare"
    priority = 0
    action { allow {} }
    statement {
      ip_set_reference_statement {
        arn = aws_wafv2_ip_set.cloudflare.arn
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "allow-cloudflare"
      sampled_requests_enabled   = true
    }
  }

  # Managed rules
  rule {
    name     = "AWS-AWSManagedRulesCommonRuleSet"
    priority = 2
    override_action { none {} }
    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesCommonRuleSet"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesCommonRuleSet"
      sampled_requests_enabled   = true
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesSQLiRuleSet"
    priority = 3
    override_action { none {} }
    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesSQLiRuleSet"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesSQLiRuleSet"
      sampled_requests_enabled   = true
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesWordPressRuleSet"
    priority = 4
    override_action { none {} }
    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesWordPressRuleSet"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesWordPressRuleSet"
      sampled_requests_enabled   = true
    }
  }

  # Custom rules
  rule {
    name     = "wp-login-rate-limit"
    priority = 10
    action { block {} }
    statement {
      rate_based_statement {
        limit              = 100
        aggregate_key_type = "IP"
        scope_down_statement {
          regex_pattern_set_reference_statement {
            arn = aws_wafv2_regex_pattern_set.wp_paths.arn
            field_to_match { uri_path {} }
            text_transformation {
              priority = 0
              type     = "NONE"
            }
          }
        }
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "wp-login-rate-limit"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "wordpress-waf"
    sampled_requests_enabled   = true
  }
}

resource "aws_wafv2_ip_set" "cloudflare" {
  name               = "cloudflare-ips"
  scope              = "REGIONAL"
  ip_address_version = "IPV4"
  addresses          = ["173.245.48.0/20", "103.21.244.0/22"]
}

resource "aws_wafv2_regex_pattern_set" "wp_paths" {
  name  = "wp-sensitive-paths"
  scope = "REGIONAL"
  regular_expression { regex = "/wp-login\\.php" }
  regular_expression { regex = "/wp-admin" }
  regular_expression { regex = "/xmlrpc\\.php" }
}

resource "aws_wafv2_web_acl_association" "wordpress" {
  resource_arn = aws_lb.wordpress.arn
  web_acl_arn  = aws_wafv2_web_acl.wordpress.arn
}
```

## Verification

- [ ] WAF Web ACL created
- [ ] Attached to ALB or CloudFront
- [ ] AWS managed rule groups enabled (Common, SQLi, PHP, WordPress)
- [ ] Rate limiting on `/wp-login.php` configured
- [ ] XML-RPC blocked
- [ ] Cloudflare IPs whitelisted (if applicable)
- [ ] Logging to CloudWatch enabled
- [ ] Dashboard created
- [ ] Alerts configured for high block rate
- [ ] Test attacks return 403
