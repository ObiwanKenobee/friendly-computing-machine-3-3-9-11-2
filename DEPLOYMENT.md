# QuantumVest Enterprise - AWS Production Deployment

Complete guide for deploying QuantumVest Enterprise to AWS production environment using modern, cost-effective architecture.

## 🏗️ **Recommended Architecture: AWS App Runner + CloudFront + RDS**

### **Why App Runner over Elastic Beanstalk?**

| Feature               | App Runner               | Elastic Beanstalk          |
| --------------------- | ------------------------ | -------------------------- |
| **Setup Complexity**  | ⭐⭐⭐⭐⭐ Simple        | ⭐⭐⭐ Moderate            |
| **Cost**              | ⭐⭐⭐⭐⭐ Pay-per-use   | ⭐⭐⭐ Always-on instances |
| **Scaling**           | ⭐⭐⭐⭐⭐ Automatic     | ⭐⭐⭐⭐ Configurable      |
| **Maintenance**       | ⭐⭐⭐⭐⭐ Fully managed | ⭐⭐⭐ Platform updates    |
| **Container Support** | ⭐⭐⭐⭐⭐ Native        | ⭐⭐⭐ Via Docker          |

## 🎯 **Production Architecture Overview**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Route 53      │───▶│   CloudFront    │───▶│   App Runner    │
│   DNS           │    │   CDN           │    │   Application   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                       ┌─────────────────┐    ┌─────────────────┐
                       │   ElastiCache   │◀───┤   RDS Aurora    │
                       │   Redis         │    │   PostgreSQL    │
                       └─────────────────┘    └─────────────────┘
```

### **Key Components:**

- **🌐 Route 53**: DNS management
- **🚀 CloudFront**: Global CDN with edge caching
- **⚡ App Runner**: Serverless container runtime
- **🗄️ RDS Aurora Serverless v2**: Auto-scaling database
- **⚡ ElastiCache Redis**: High-performance caching
- **📦 S3**: Static asset storage
- **📊 CloudWatch**: Monitoring and alerting

## 📋 **Prerequisites**

### **Required Tools:**

```bash
# AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Node.js 18+
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify versions
aws --version    # Should be 2.x
node --version   # Should be v18.x or higher
npm --version    # Should be 8.x or higher
```

### **AWS Account Setup:**

1. **Domain & SSL Certificate** (Required):

   ```bash
   # Request SSL certificate in us-east-1 (required for CloudFront)
   aws acm request-certificate \
     --domain-name quantumvest.app \
     --subject-alternative-names www.quantumvest.app \
     --validation-method DNS \
     --region us-east-1
   ```

2. **Route 53 Hosted Zone** (Required):

   ```bash
   # Create hosted zone for your domain
   aws route53 create-hosted-zone \
     --name quantumvest.app \
     --caller-reference $(date +%s)
   ```

3. **AWS IAM Permissions** (Required):
   - CloudFormation (full access)
   - App Runner (full access)
   - RDS (full access)
   - ElastiCache (full access)
   - S3 (full access)
   - CloudFront (full access)
   - Route 53 (full access)
   - IAM (limited for role creation)

## 🚀 **One-Click Production Deployment**

### **Step 1: Environment Configuration**

```bash
# Copy environment template
cp .env.production.template .env.production

# Edit with your actual values
nano .env.production
```

**Required Environment Variables:**

```bash
# AWS Infrastructure
CERTIFICATE_ARN=arn:aws:acm:us-east-1:123456789012:certificate/your-cert-id
DOMAIN_NAME=quantumvest.app
GITHUB_REPO=https://github.com/your-org/quantumvest

# Core Services (REQUIRED)
VITE_OPENAI_API_KEY=sk-prod-your_key_here
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your_anon_key

# Payment Processing
VITE_STRIPE_PUBLIC_KEY=pk_live_your_stripe_key
VITE_PAYPAL_CLIENT_ID=your_paypal_client_id

# Analytics
VITE_GOOGLE_ANALYTICS_ID=G-YOUR_GA_ID
VITE_SENTRY_DSN=https://your_sentry_dsn@sentry.io/project
```

### **Step 2: Deploy to Production**

```bash
# Make deployment script executable
chmod +x scripts/deploy-aws-production.sh

# Deploy everything
./scripts/deploy-aws-production.sh
```

The script will:

1. ✅ **Validate prerequisites** (AWS CLI, Node.js, credentials)
2. 🔧 **Build and test** the application
3. 🏗️ **Deploy infrastructure** via CloudFormation
4. 🚀 **Deploy application** to App Runner
5. ✅ **Verify deployment** with health checks
6. 📊 **Setup monitoring** and alerting
7. 🔒 **Apply security hardening**

### **Step 3: DNS Configuration**

```bash
# Get your Route 53 name servers
aws route53 get-hosted-zone --id /hostedzone/YOUR_ZONE_ID

# Update your domain registrar to use Route 53 name servers
# Example name servers:
# ns-123.awsdns-12.com
# ns-456.awsdns-45.net
# ns-789.awsdns-78.org
# ns-012.awsdns-01.co.uk
```

## 💰 **Cost Optimization Features**

### **1. Auto-Scaling Architecture**

- **App Runner**: Scales to zero when no traffic
- **RDS Aurora Serverless v2**: Scales from 0.5 ACU to 4 ACU
- **ElastiCache**: Right-sized instances with reserved capacity

### **2. Estimated Monthly Costs (USD)**

| Component         | Low Traffic | Medium Traffic | High Traffic |
| ----------------- | ----------- | -------------- | ------------ |
| **App Runner**    | $5-15       | $25-50         | $100-200     |
| **RDS Aurora**    | $15-30      | $50-100        | $200-400     |
| **ElastiCache**   | $15-25      | $25-50         | $50-100      |
| **CloudFront**    | $1-5        | $10-25         | $50-100      |
| **S3 + Route 53** | $2-5        | $5-10          | $10-25       |
| **Total**         | **$38-80**  | **$115-235**   | **$410-825** |

### **3. Cost Monitoring Alarms**

```bash
# Set up billing alerts
aws budgets create-budget \
  --account-id $(aws sts get-caller-identity --query Account --output text) \
  --budget file://aws/monitoring/budget-alert.json
```

## 📊 **Monitoring & Alerting**

### **1. CloudWatch Dashboard**

Access your dashboard: `https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#dashboards:name=QuantumVest-production`

**Key Metrics:**

- **App Runner**: Request count, latency, error rate
- **RDS**: CPU, connections, storage
- **CloudFront**: Cache hit ratio, data transfer
- **Redis**: CPU, memory, connections

### **2. Performance Targets**

- **Response Time**: < 2 seconds (95th percentile)
- **Availability**: 99.9% uptime
- **Error Rate**: < 0.1%
- **Cache Hit Ratio**: > 90%

### **3. Alerting Channels**

```bash
# Configure SNS for alerts
aws sns create-topic --name quantumvest-production-alerts
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:quantumvest-production-alerts \
  --protocol email \
  --notification-endpoint your-email@company.com
```

## 🔒 **Security & Compliance**

### **1. Built-in Security Features**

- ✅ **WAF Protection**: Enabled on CloudFront
- ✅ **DDoS Protection**: AWS Shield Standard
- ✅ **SSL/TLS**: End-to-end encryption
- ✅ **VPC Isolation**: Database in private subnets
- ✅ **IAM Roles**: Least privilege access
- ✅ **Encryption**: At rest and in transit

### **2. Compliance Standards**

- ✅ **SOC 2 Type II**: AWS services are compliant
- ✅ **PCI DSS**: For payment processing
- ✅ **GDPR**: Data protection controls
- ✅ **CCPA**: California privacy compliance

### **3. Security Monitoring**

```bash
# Enable GuardDuty (automated threat detection)
aws guardduty create-detector --enable --region us-east-1

# Enable AWS Config (compliance monitoring)
aws configservice put-configuration-recorder \
  --configuration-recorder name=default \
  --recording-group allSupported=true
```

## 🚀 **Advanced Features**

### **1. Blue/Green Deployments**

```bash
# Deploy to staging first
ENVIRONMENT=staging ./scripts/deploy-aws-production.sh

# Test staging environment
curl -f https://staging.quantumvest.app/api/health

# Promote to production
ENVIRONMENT=production ./scripts/deploy-aws-production.sh
```

### **2. Database Migrations**

```bash
# Run database migrations
npm run db:migrate

# Seed production data
npm run db:seed
```

### **3. Performance Optimization**

```bash
# Enable CloudFront compression
aws cloudfront update-distribution \
  --id $CLOUDFRONT_DISTRIBUTION_ID \
  --distribution-config file://aws/cloudfront/compression-config.json

# Setup ElastiCache for API responses
# Automatically configured in the CloudFormation template
```

## 🔧 **Troubleshooting**

### **Common Issues:**

1. **Certificate Validation Failed**

   ```bash
   # Check certificate status
   aws acm describe-certificate --certificate-arn $CERTIFICATE_ARN
   # Ensure DNS validation is complete
   ```

2. **App Runner Build Failed**

   ```bash
   # Check build logs
   aws apprunner describe-service --service-arn $SERVICE_ARN
   # View detailed logs in CloudWatch
   ```

3. **Database Connection Issues**

   ```bash
   # Check security groups
   aws ec2 describe-security-groups --group-ids $DB_SECURITY_GROUP_ID
   # Verify VPC connectivity
   ```

4. **High Costs**
   ```bash
   # Check AWS Cost Explorer
   aws ce get-cost-and-usage \
     --time-period Start=2023-01-01,End=2023-01-31 \
     --granularity DAILY \
     --metrics BlendedCost
   ```

### **Performance Debugging:**

```bash
# Check App Runner metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/AppRunner \
  --metric-name RequestLatency \
  --dimensions Name=ServiceName,Value=production-quantumvest-app \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 3600 \
  --statistics Average

# Monitor database performance
aws rds describe-db-cluster-parameters \
  --db-cluster-parameter-group-name default.aurora-postgresql15
```

## 📞 **Support & Maintenance**

### **1. Backup Strategy**

- **Database**: Automated daily backups (30-day retention)
- **S3 Assets**: Versioning enabled with lifecycle policies
- **Configuration**: Infrastructure as Code in Git

### **2. Update Process**

```bash
# Update application code
git push origin main  # App Runner auto-deploys

# Update infrastructure
./scripts/deploy-aws-production.sh  # Re-run deployment

# Update dependencies
npm update && npm audit fix
```

### **3. Monitoring Health**

```bash
# Quick health check
curl -f https://quantumvest.app/api/health

# Detailed monitoring
aws cloudwatch get-dashboard \
  --dashboard-name QuantumVest-production
```

## 🎉 **Success Metrics**

After successful deployment, you should see:

- ✅ **Website**: https://quantumvest.app (loads in < 2 seconds)
- ✅ **API Health**: https://quantumvest.app/api/health (returns 200 OK)
- ✅ **SSL Rating**: A+ on SSL Labs test
- ✅ **Performance**: 90+ on Google PageSpeed Insights
- ✅ **Uptime**: 99.9% availability
- ✅ **Cost**: Optimized for traffic patterns

---

## 📖 **Additional Resources**

- [AWS App Runner Documentation](https://docs.aws.amazon.com/apprunner/)
- [CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [QuantumVest API Documentation](./API_DOCUMENTATION.md)

**Need help?** Contact the platform team or create an issue in the repository.
