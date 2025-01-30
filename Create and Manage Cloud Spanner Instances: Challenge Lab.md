## Environment Setup

- 🌍 Set the region:
  ```sh
  export REGION=europe-west1  
  export BUCKET_NAME=qwiklabs-gcp-01-026491e087be
  ```

## Create Spanner Instance

🏩 Create the Spanner instance:

```sh
gcloud spanner instances create banking-ops-instance \
    --config=regional-$REGION \
    --description="Banking Operations Instance" \
    --nodes=1
```

🗄 Create the Spanner database:

```sh
gcloud spanner databases create banking-ops-db \
    --instance=banking-ops-instance
```

## Create Tables

📊 Create the Portfolio table:

```sql
CREATE TABLE Portfolio (
  PortfolioId INT64 NOT NULL,
  Name STRING(MAX),
  ShortName STRING(MAX),
  PortfolioInfo STRING(MAX)
) PRIMARY KEY (PortfolioId);
```

📊 Create the Category table:

```sql
CREATE TABLE Category (
  CategoryId INT64 NOT NULL,
  PortfolioId INT64 NOT NULL,
  CategoryName STRING(MAX),
  PortfolioInfo STRING(MAX)
) PRIMARY KEY (CategoryId);
```

📊 Create the Product table:

```sql
CREATE TABLE Product (
  ProductId INT64 NOT NULL,
  CategoryId INT64 NOT NULL,
  PortfolioId INT64 NOT NULL,
  ProductName STRING(MAX),
  ProductAssetCode STRING(25),
  ProductClass STRING(25)
) PRIMARY KEY (ProductId);
```

📊 Create the Customer table:

```sql
CREATE TABLE Customer (
  CustomerId STRING(36) NOT NULL,
  Name STRING(MAX) NOT NULL,
  Location STRING(MAX) NOT NULL
) PRIMARY KEY (CustomerId);
```

## Insert Data

🌱 Insert data into Portfolio:

```sql
INSERT INTO Portfolio (PortfolioId, Name, ShortName, PortfolioInfo) VALUES
  (1, "Banking", "Bnkg", "All Banking Business"),
  (2, "Asset Growth", "AsstGrwth", "All Asset Focused Products"),
  (3, "Insurance", "Insurance", "All Insurance Focused Products");
```

🌱 Insert data into Category:

```sql
INSERT INTO Category (CategoryId, PortfolioId, CategoryName, PortfolioInfo) VALUES
  (1, 1, "Cash", NULL),
  (2, 2, "Investments - Short Return", NULL),
  (3, 2, "Annuities", NULL),
  (4, 3, "Life Insurance", NULL);
```

🌱 Insert data into Product:

```sql
INSERT INTO Product (ProductId, CategoryId, PortfolioId, ProductName, ProductAssetCode, ProductClass) VALUES
  (1, 1, 1, "Checking Account", "ChkAcct", "Banking LOB"),
  (2, 2, 2, "Mutual Fund Consumer Goods", "MFundCG", "Investment LOB"),
  (3, 3, 2, "Annuity Early Retirement", "AnnuFixed", "Investment LOB"),
  (4, 4, 3, "Term Life Insurance", "TermLife", "Insurance LOB"),
  (5, 1, 1, "Savings Account", "SavAcct", "Banking LOB"),
  (6, 1, 1, "Personal Loan", "PersLn", "Banking LOB"),
  (7, 1, 1, "Auto Loan", "AutLn", "Banking LOB"),
  (8, 4, 3, "Permanent Life Insurance", "PermLife", "Insurance LOB"),
  (9, 2, 2, "US Savings Bonds", "USSavBond", "Investment LOB");
```

## Data Flow and Storage

📂 Copy the CSV file:

```sh
gsutil cp gs://cloud-training/OCBL375/Customer_List_500.csv .
```

🚀 Enable necessary services:

```sh
gcloud services enable dataflow.googleapis.com
```

```sh
gcloud services enable spanner.googleapis.com
```

## Manifest and Data Import

📝 Create the manifest file:

```sh
cat > manifest.json << EOF_CP
{
  "tables": [
    {
      "table_name": "Customer",
      "file_patterns": [
        "gs://cloud-training/OCBL375/Customer_List_500.csv"
      ],
      "columns": [
        {"column_name" : "CustomerId", "type_name" : "STRING" },
        {"column_name" : "Name", "type_name" : "STRING" },
        {"column_name" : "Location", "type_name" : "STRING" }
      ]
    }
  ]
}
EOF_CP
```

📂 Create a storage bucket and upload the manifest:

```sh
gcloud storage buckets create gs://$BUCKET_NAME --location=$REGION
```

```sh
gsutil cp manifest.json gs://$BUCKET_NAME
```

🚀 Run the Dataflow job:

```sh
gcloud dataflow jobs run spanner-import-job \
    --gcs-location=gs://dataflow-templates/latest/GCS_Text_to_Cloud_Spanner \
    --region=$REGION \
    --parameters=instanceId="banking-ops-instance",databaseId="banking-ops-db",importManifest="gs://$BUCKET_NAME/manifest.json"
```

## Database Schema Updates

🔧 Add a new column to the Category table:

```sh
gcloud spanner databases ddl update banking-ops-db \
--instance=banking-ops-instance \
--ddl="ALTER TABLE Category ADD COLUMN MarketingBudget INT64;"
```

📝 Describe the database schema:

```sh
gcloud spanner databases ddl describe banking-ops-db --instance=banking-ops-instance
```
