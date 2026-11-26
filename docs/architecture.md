# Architecture

Three-tier app in a VPC: public web tier, private database tier, no NAT gateway.

```mermaid
flowchart TB
    users([Users and browsers])

    subgraph aws["AWS Cloud - us-east-1"]
        igw["Internet gateway"]

        subgraph vpc["VPC shop-vpc - 10.0.0.0/16"]
            subgraph azA["Availability zone A"]
                subgraph pubA["Public subnet - route 0.0.0.0/0 to IGW"]
                    web["EC2 web-1<br/>t3.micro, nginx<br/>security group web-sg"]
                end
                subgraph privA["Private subnet - local routes only"]
                    rds[("RDS MySQL<br/>db.t3.micro, single-AZ<br/>security group db-sg")]
                end
            end

            subgraph azB["Availability zone B"]
                pubB["Public subnet<br/>spare, for scaling out"]
                privB["Private subnet<br/>needed by the DB subnet group"]
            end

            s3ep["S3 gateway endpoint<br/>free"]
        end
    end

    users -->|"HTTP 80"| igw
    igw --> web
    web -->|"MySQL 3306"| rds
```

## Security groups

| Group | Inbound rule | Source |
|-------|--------------|--------|
| web-sg | HTTP, port 80 | Anywhere (0.0.0.0/0) |
| db-sg | MySQL, port 3306 | web-sg only |

## Design choices

- No NAT gateway, which saves about $30 a month. The database never needs the internet.
- The database has no public address, so it is reachable only from the web tier.
- The second availability zone is required by RDS subnet groups and leaves room to add a second web server.
- No SSH. The web server is managed through Session Manager.
