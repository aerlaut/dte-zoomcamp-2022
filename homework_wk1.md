# Week 1 Homework

Wk1 homework code snippets :

### Q1 - Google cloud version

To get the Google Cloud version, I ran

    gcloud --version

Which outputs

    Google Cloud SDK 369.0.0
    bq 2.0.72
    core 2022.01.14
    gsutil 5.6

The first line shows tha the GCloud SDK version is version `369.0.0`.

### Q2 - Output of `terraform apply`

To answer question no 2, I ran `terraform apply` from inside the `2_docker_sql` folder, and supplying it with my project ID. The output of the command is as follows :

    var.project
    Your GCP Project ID

    Enter a value: driven-nature-338814

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols: + create

Terraform will perform the following actions:

    # google_bigquery_dataset.dataset will be created
    + resource "google_bigquery_dataset" "dataset" {
        + creation_time              = (known after apply)
        + dataset_id                 = "trips_data_all"
        + delete_contents_on_destroy = false
        + etag                       = (known after apply)
        + id                         = (known after apply)
        + last_modified_time         = (known after apply)
        + location                   = "europe-west6"
        + project                    = "driven-nature-338814"
        + self_link                  = (known after apply)

        + access {
            + domain         = (known after apply)
            + group_by_email = (known after apply)
            + role           = (known after apply)
            + special_group  = (known after apply)
            + user_by_email  = (known after apply)

            + view {
                + dataset_id = (known after apply)
                + project_id = (known after apply)
                + table_id   = (known after apply)
              }
          }
      }

    # google_storage_bucket.data-lake-bucket will be created
    + resource "google_storage_bucket" "data-lake-bucket" {
        + force_destroy               = true
        + id                          = (known after apply)
        + location                    = "EUROPE-WEST6"
        + name                        = "dtc_data_lake_driven-nature-338814"
        + project                     = (known after apply)
        + self_link                   = (known after apply)
        + storage_class               = "STANDARD"
        + uniform_bucket_level_access = true
        + url                         = (known after apply)

        + lifecycle_rule {
            + action {
                + type = "Delete"
              }

            + condition {
                + age                   = 30
                + matches_storage_class = []
                + with_state            = (known after apply)
              }
          }

        + versioning {
            + enabled = true
          }
      }

      Plan: 2 to add, 0 to change, 0 to destroy.

      Do you want to perform these actions?
        Terraform will perform the actions described above.
        Only 'yes' will be accepted to approve.

        Enter a value: yes

      google_bigquery_dataset.dataset: Creating...
      google_storage_bucket.data-lake-bucket: Creating...
      google_storage_bucket.data-lake-bucket: Creation complete after 1s [id=dtc_data_lake_driven-nature-338814]
      google_bigquery_dataset.dataset: Creation complete after 1s [id=projects/driven-nature-338814/datasets/trips_data_all]

      Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

### Q3 - Number of trips in 15 Jan

SQL query to get the number of trips in Jan 15 :

    SELECT COUNT(*) as num_trips
    FROM yellow_taxi_trips
    WHERE CAST(tpep_pickup_datetime AS DATE) = '2021-01-15'

### Q4 - Date with the largest tip

SQL query to get the largest tip per day

    SELECT CAST(tpep_pickup_datetime AS DATE) as pickup_date, MAX(tip_amount) as max_tip
    FROM yellow_taxi_trips
    GROUP BY pickup_date
    ORDER BY max_tip DESC
    LIMIT 1

### Q5 - Most popular destination from Central Park

SQL query to get the most popular destination from Central Park

    SELECT "Zone", num_trips from (
      SELECT "DOLocationID", COUNT(*) as num_trips
      FROM yellow_taxi_trips
      WHERE CAST(tpep_pickup_datetime AS DATE) = '2021-01-14'
      AND "PULocationID" = (
        SELECT "LocationID" FROM zone_lookup WHERE "Zone" = 'Central Park'
      )
      GROUP BY "DOLocationID"
      ORDER BY num_trips DESC
    ) as trips
    INNER JOIN zone_lookup
    ON trips."DOLocationID" = zone_lookup."LocationID"
    ORDER BY num_trips DESC
    LIMIT 1

### Q6 - Trip pair with the highest average trip amount

SQL query to get the trip pair with the highest average amount

    SELECT
      do_location_tbl."Zone" as do_location,
      pu_location_tbl."Zone" as pu_location,
      avg_amount
    FROM (
      SELECT
        "DOLocationID",
        "PULocationID",
        AVG(total_amount) as avg_amount
      FROM yellow_taxi_trips
      GROUP BY "DOLocationID", "PULocationID"
      ) as trips_fare
    INNER JOIN zone_lookup AS do_location_tbl
    ON trips_fare."DOLocationID" = do_location_tbl."LocationID"
    INNER JOIN zone_lookup AS pu_location_tbl
    ON trips_fare."PULocationID" = pu_location_tbl."LocationID"
    ORDER BY avg_amount DESC
    LIMIT 1
