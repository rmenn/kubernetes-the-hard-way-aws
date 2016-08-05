#### Infra Setup 
This goes thru the basic setup which is required. 

###### Creating the VPC
```
~|⇒ aws ec2 create-vpc --cidr-block 10.4.0.0/16
{
    "Vpc": {
        "VpcId": "vpc-957391f1",
        "InstanceTenancy": "default",
        "State": "pending",
        "DhcpOptionsId": "dopt-03111b61",
        "CidrBlock": "10.4.0.0/16",
        "IsDefault": false
    }
}
```

###### Enabling DNS for the VPC
```
~|⇒ aws ec2 modify-vpc-attribute --vpc-id vpc-957391f1 --enable-dns-support
~|⇒ aws ec2 modify-vpc-attribute --vpc-id vpc-957391f1 --enable-dns-hostnames
```

###### Creating Subnets
Creating 2 subnets one for the master and one for the workers
```
~|⇒ aws ec2 create-subnet --vpc-id vpc-957391f1 --cidr-block 10.4.1.0/24
{
    "Subnet": {
        "VpcId": "vpc-957391f1",
        "CidrBlock": "10.4.1.0/24",
        "State": "pending",
        "AvailabilityZone": "ap-southeast-1a",
        "SubnetId": "subnet-ecd3049a",
        "AvailableIpAddressCount": 251
    }
}
~|⇒ aws ec2 create-subnet --vpc-id vpc-957391f1 --cidr-block 10.4.2.0/24
{
    "Subnet": {
        "VpcId": "vpc-957391f1",
        "CidrBlock": "10.4.2.0/24",
        "State": "pending",
        "AvailabilityZone": "ap-southeast-1a",
        "SubnetId": "subnet-d4d304a2",
        "AvailableIpAddressCount": 251
    }
}
```
###### Internetgateway
```
~|⇒ aws ec2 create-internet-gateway
{
    "InternetGateway": {
        "Tags": [],
        "InternetGatewayId": "igw-cceb88a9",
        "Attachments": []
    }
}
```
###### Attaching the Internet gatewat to the VPC
```
~|⇒ aws ec2 attach-internet-gateway --internet-gateway-id igw-cceb88a9 --vpc-id vpc-957391f1
```
```
###### Route Table
~|⇒ aws ec2 create-route-table --vpc-id vpc-957391f1
{
    "RouteTable": {
        "Associations": [],
        "RouteTableId": "rtb-9af728fe",
        "VpcId": "vpc-957391f1",
        "PropagatingVgws": [],
        "Tags": [],
        "Routes": [
            {
                "GatewayId": "local",
                "DestinationCidrBlock": "10.4.0.0/16",
                "State": "active",
                "Origin": "CreateRouteTable"
            }
        ]
    }
}
```
###### Route Table Association
```
~|⇒ aws ec2 associate-route-table --route-table-id rtb-9af728fe --subnet-id subnet-ecd3049a
{
    "AssociationId": "rtbassoc-725e3e16"
}
~|⇒ aws ec2 associate-route-table --route-table-id rtb-9af728fe --subnet-id subnet-d4d304a2
{
    "AssociationId": "rtbassoc-405e3e24"
}
```
###### Security groups and Rules
```
~|⇒ aws ec2 create-security-group --vpc-id vpc-957391f1 --group-name workers --description k8s-workers
{
    "GroupId": "sg-ce95c2aa"
}
~|⇒ aws ec2 create-security-group --vpc-id vpc-957391f1 --group-name master --description k8s-master
{
    "GroupId": "sg-d695c2b2"
}
~|⇒ aws ec2 authorize-security-group-ingress --group-id sg-d695c2b2 --port 0-65535 --protocol tcp --source-group sg-ce95c2aa
~|⇒ aws ec2 authorize-security-group-ingress --group-id sg-ce95c2aa --port 0-65535 --protocol tcp --source-group sg-d695c2b2
```
###### Run instances
```
`aws ec2 run-instances --image-id <> --count 3 --instance-type t2.micro --key-name k8s-test --security-group-ids <> --subnet-id <> --associate-public-ip-address`

`aws ec2 run-instances --image-id <> --count 1 --instance-type t2.micro --key-name k8s-test --security-group-ids <> --subnet-id <> --associate-public-ip-address`
```
