# Local development with DAB (Data API Builder)
> Using Docker, SQL Server, .NET 6.0 and Data API Builder (DAB)

## Prerequisites

- [Create a workspaces environment (Optional)](https://github.com/features/codespaces)
- [Install SqlCmd (Go)](https://learn.microsoft.com/en-us/sql/tools/sqlcmd/sqlcmd-utility?view=sql-server-ver16&tabs=go%2Clinux&pivots=cs1-bash#tabpanel_2_go)
- [Install .NET 6.0](https://dotnet.microsoft.com/en-us/download/dotnet/6.0)
- [Install Data API Builder (DAB)](https://docs.microsoft.com/en-us/sql/relational-databases/data-api/install-run-dab?view=sql-server-ver15#install-dab)

Here you have the instructions to install the prerequisites in Ubuntu 20.04 with GitHub Codespaces:

## Installation
### 1. Install SqlCmd (Go) - Ubuntu 20.04

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/20.04/prod.list)"
sudo apt-get update
sudo  apt-get install sqlcmd
```

### 2. Install .NET 6.0 - Ubuntu 20.04

```bash
wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh
chmod +x ./dotnet-install.sh
./dotnet-install.sh --version latest
./dotnet-install.sh --version latest --runtime aspnetcore
dotnet --list-sdks
```

### 3. Install Data API Builder (DAB) - Ubuntu 20.04

```bash
dotnet tool install --global Microsoft.DataApiBuilder
dotnet tool update --global Microsoft.DataApiBuilder
echo 'export PATH="/usr/local/dotnet/7.0.306/tools:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## Build the backend

### Create a SQL Server Container a backup
> This example uses the AdventureWorks light version, you can use your own backup (from a URL)

```bash
# Create SQL Database from a backup
sqlcmd create mssql --accept-eula --using https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorksLT2022.bak

# Get connection strings
sqlcmd config connection-strings
```

### Use DAB to create the backend (REST and GraphQL endpoints)

> Do not forget to update the connection string in the dab-config.json file

```bash
# Create a new DAB project (config file)
dab init --database-type mssql  --config "dab-config.json" --host-mode Development

# Modify dab-config.json with connection string
dab add Customer --config dab-config.json --source SalesLT.Customer --permissions "anonymous:*"

# Start DAB's engine
dab start --config dab-config.json
```

### Test the backend

Use the following endpoints to test the *REST API* backend:

```bash
# Get all customers
GET https://127.0.0.1:5001/api/Customer

# Get customer details when customer id = 1**
GET https://127.0.0.1:5001/api/Customer/CustomerID/1 

# Get customers first and last name**
GET https://127.0.0.1:5001/api/Customer?$select=FirstName,LastName

# Get customers details when fist name = Orlando
GET https://127.0.0.1:5001/api/Customer?$filter=FirstName eq 'Orlando'

# Get customers first and last name. Ordered by first name descending
GET https://127.0.0.1:5001/api/Customer?$orderby=FirstName desc ,LastName

# Get first five customers
GET https://127.0.0.1:5001/api/Customer?$first=5

# Update customer's info
PATCH https://127.0.0.1:5001/api/Customer/CustomerID/1 
Content-type: application/json

{  
  "FirstName": "Rick",
  "LastName": "Deckard"
}
```