# Cloudfront using multiple origins

Example of multiple target origins

```
resource "aws_cloudfront_distribution" "example" {
  enabled = true
  origins {
    domain_name = aws_apigatewayv2_api.example.api_endpoint
      # Strip https:// and path!
    origin_id   = "api-gateway-origin"

    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }

  # Default behavior (e.g., root path or catch-all)
  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "api-gateway-origin"
    viewer_protocol_policy = "redirect-to-https"

    forwarded_values {
      query_string = true
      headers      = ["Authorization"]
      cookies {
        forward = "none"
      }
    }
  }

  # Specific path behavior for /api/*
  ordered_cache_behavior {
    path_pattern     = "api/*"
    allowed_methods  = ["GET", "HEAD", "OPTIONS", "POST"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "api-gateway-origin"

    viewer_protocol_policy = "redirect-to-https"

    forwarded_values {
      query_string = true
      headers      = ["Authorization"]
      cookies {
        forward = "none"
      }
    }
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
    # or use acm_certificate_arn if using a custom domain and ACM
  }
}

```