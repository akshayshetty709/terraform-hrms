# HRMS Backend - EC2 Infrastructure (Terraform)

Provisions a single EC2 instance (t3.xlarge, 100GB gp3) in a new VPC in
`ap-south-1`, with Docker + Docker Compose pre-installed via user data,
ready to run your `docker-compose.yml` stack (Kafka, Zookeeper, Postgres,
Redis, pgadmin, schema-registry, and your microservices).

## Prerequisites

- Terraform >= 1.5.0
- AWS credentials configured (`aws configure` or env vars)
- An **existing EC2 key pair** already created in the `ap-south-1` region

## Setup

1. Copy the example vars file and fill in your key pair name and your IP:
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   ```
   Edit `terraform.tfvars`:
   - `key_name` — the name of your existing EC2 key pair
   - `allowed_ssh_cidr` — restrict this to your actual IP (e.g. `203.0.113.5/32`)
     instead of leaving it open to the world

2. Initialize and apply:
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

3. Once applied, Terraform prints the public IP and an SSH command:
   ```bash
   terraform output ssh_command
   ```

4. SSH in, clone/copy your repo, and run your stack:
   ```bash
   ssh -i /path/to/your-key.pem ubuntu@<public-ip>
   git clone <your-repo>
   cd Backend
   docker compose up -d --build
   ```
   Docker and the Compose plugin are already installed by the user-data
   script, along with a `daemon.json` DNS fix (8.8.8.8 / 1.1.1.1) to avoid
   the `npm ci ETIMEDOUT` registry issue you hit earlier.

## What this creates

| Resource | Purpose |
|---|---|
| VPC + public subnet + IGW + route table | Isolated network for the instance |
| Security group | Opens SSH (restricted) + app ports (8080, 3000, 5050 by default) |
| EC2 instance (t3.xlarge, 100GB gp3) | Runs Docker + your compose stack |
| Elastic IP | Stable public IP that survives instance stop/start |

## Customizing

- **App ports**: edit `app_ports` in `terraform.tfvars` to match whatever
  your services actually expose (e.g. add 5432 for direct Postgres access,
  9092 for Kafka, etc. — only add what you actually need publicly reachable).
- **Instance size**: change `instance_type` if t3.xlarge is overkill or not
  enough once you measure real usage (`docker stats` on the running box).
- **Jenkins**: if Jenkins itself will also run on this box, add port 8080
  is already included by default for that reason.

## Destroying

```bash
terraform destroy
```
This terminates the instance and tears down the VPC/networking — nothing
persists (no S3 state for your app data), so make sure your Postgres data
is backed up or on a volume you manage separately if this isn't just a
disposable dev/staging box.
