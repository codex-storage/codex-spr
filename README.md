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
    networks_file="codex"
    buckets="bucket-1 bucket-2"

    for bucket in ${buckets}; do

      # Records
      for file in *.txt; do
        # Variables
        network=$(awk -F '-' '{print $1}' <<< ${file})
        nodes=$(awk -F '-' '{print $2}' <<< ${file%%.*})
        echo "${network}" >> "${networks_file}"

        # Upload
        if [[ "${nodes}" == "codex" ]]; then
          aws s3 cp --acl public-read --content-type "text/plain" "${file}" "s3://${bucket}/${network}"
        fi
        aws s3 cp --acl public-read --content-type "text/plain" "${file}" "s3://${bucket}/${network}/${nodes}"
      done

      # Networks
      sort -u "${networks_file}" -o "${networks_file}"
      aws s3 cp --acl public-read --content-type "text/plain" "${networks_file}" "s3://${bucket}/${networks_file}"

    done

    rm -f "${networks_file}"
    ```

 4. Check the result
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
