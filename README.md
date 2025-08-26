# Codex SPR records

 Repository contains Codex nodes SPR records and ENR records for Testnet Geth nodes.

 Records also can be found in the [docs](https://docs.codex.storage/networks/networks).


## Get records

```shell
# Networks
curl https://spr.codex.storage

# Codex
curl https://spr.codex.storage/testnet
curl https://spr.codex.storage/testnet/codex
curl https://spr.codex.storage/devnet/codex

# Geth
curl https://spr.codex.storage/testnet/geth
```


## Update SPR records

 1. Update files with SPR/ENR records or add new ones for the new network in the following format
    ```shell
    <network>-<nodes>.txt
    ```

 2. Authenticate on AWS
    ```shell
    export AWS_ACCESS_KEY_ID="<access key>"
    export AWS_SECRET_ACCESS_KEY="<secret access key>"
    export AWS_DEFAULT_REGION="eu-central-1"
    ```

 3. Update files to S3
    ```shell
    # Variables
    networks_file="codex"
    buckets="bucket-1 bucket-2"
    folder="upload"


    # Networks records
    mkdir -p "${folder}"
    for file in *.txt; do

      # Variables
      network=$(awk -F '-' '{print $1}' <<< ${file})
      nodes=$(awk -F '-' '{print $2}' <<< ${file%%.*})
      echo "${network}" >> "${networks_file}"

      # Networks folders/files
      mkdir -p "${folder}/${network}"
      cp "${file}" "${folder}/${network}/${nodes}"
    done

    # Networks list
    sort -u "${networks_file}" -o "${folder}/${networks_file}"

    # Upload to S3
    for bucket in ${buckets}; do

      # Sync networks
      aws s3 sync --acl public-read --content-type "text/plain" "${folder}" "s3://${bucket}" --delete

      # Copy main node
      for file in ${folder}/*/${networks_file}; do
        network=$(awk -F '/' '{print $2}' <<< "${file}")
        aws s3 cp --acl public-read --content-type "text/plain" "${file}" "s3://${bucket}/${network}"
      done
    done

    # Cleanup
    rm -rf "${folder}"
    ```

 4. Flush cache
    ```shell
    # Variables
    distribution_id="XXXXXXXXXXXXXX"

    # Flush cache
    invalidation_id=$(aws cloudfront create-invalidation --distribution-id "${distribution_id}" --paths "/*" | jq -r '.Invalidation.Id')
    aws cloudfront wait invalidation-completed --distribution-id "${distribution_id}" --id "${invalidation_id}"
    ```

 5. Check the result
    ```shell
    # Networks
    curl https://spr.codex.storage

    # Codex
    curl https://spr.codex.storage/testnet
    curl https://spr.codex.storage/testnet/codex

    # Geth
    curl https://spr.codex.storage/testnet/geth
    ```


## Todo

 1. Configure GitHub Actions to update records automatically.
