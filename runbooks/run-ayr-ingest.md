# Running AYR Ingest Step Function

## CLI

* Copy a bag archive to a s3 location; e.g.:

    ```bash
    # Get an arbitrary example bag
    BAG_FILE='TDR-2022-D6WD.tar.gz'
  
    BAG_URL='https://github.com/nationalarchives/'\
    'da-ayr-design-documentation/blob/main/'\
    'sample-data/bagit/'"${BAG_FILE:?}"'?raw=true'
  
    curl \
      --location \
      --output "${BAG_FILE:?}" \
      "${BAG_URL:?}"
    
    # Upload to s3
    export AWS_PROFILE=''
    S3_BUCKET=''
    
    aws s3 cp "${BAG_FILE:?}" "s3://${S3_BUCKET:?}/tre-data/${BAG_FILE:?}"
    
    # List items in path tre-data:
    aws s3 ls "s3://${S3_BUCKET:?}/tre-data/"
    ```

    ```bash
    # To remove all items in path tre-data:
    export AWS_PROFILE=''
    aws s3 rm --recursive "s3://${S3_BUCKET:?}/tre-data/"
    ```

* Create mock TRE output event with pre-signed URL to fetch TRE bag file:

    ```bash
    export AWS_PROFILE=''
    S3_BUCKET=''
    BAG_FILE='TDR-2022-D6WD.tar.gz'
    S3_URL="s3://${S3_BUCKET:?}/tre-data/${BAG_FILE:?}"
  
    TRE_OUTPUT_EVENT="$(
      printf '{
        "dri-preingest-sip-available": {
          "bag-url": "%s"
        }
      }\n' \
        "$(aws s3 presign --expires-in 60 "${S3_URL:?}")"
    )"
    
    echo "${TRE_OUTPUT_EVENT:?}"
    ```

* Run Step Function (asynchronously):

    ```bash
    export AWS_PROFILE=''
    TRE_OUTPUT_EVENT=''  # see above example
    STEP_FUNCTION_ARN=''
    EXECUTION_PREFIX='aws-cli-'
    
    # Asynchronous invocation
    SF_EXECUTION_JSON="$(
      aws \
        stepfunctions \
          start-execution \
          --state-machine-arn "${STEP_FUNCTION_ARN:?}" \
          --name "${EXECUTION_PREFIX:?}$(date +%s)" \
          --input "${TRE_OUTPUT_EVENT:?}" \
    )" && echo "${SF_EXECUTION_JSON:?}"

    # Get execution ID
    EXECUTION_ID="$(
      echo "${SF_EXECUTION_JSON:?}" \
      | python3 -c 'import json,sys;print(json.load(sys.stdin)["executionArn"])'
    )" && echo "${EXECUTION_ID:?}"
    
    # Get execution result (may still be running)
    RESULT="$(
      aws \
        stepfunctions \
          describe-execution \
          --execution-arn "${EXECUTION_ID:?}"
    )" && echo "${RESULT:?}"
    
    # Extract status field
    echo "${RESULT:?}" \
    | python3 -c 'import json,sys;print(json.load(sys.stdin, strict=False)["status"])'
    
    # Extract output field 
    echo "${RESULT:?}" \
    | python3 -c 'import json,sys;print(json.load(sys.stdin, strict=False)["output"])'
    ```
