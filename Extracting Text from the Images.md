## Prepare the application

1. Copy below script and paste it in the Cloud Shell. Before hitting the enter, change the bucket name (In order to set a unique name use your project ID because it is unique. For example, “image_bucket_YOUR_PROJECT_ID” can be your unique bucket name. Or feel free to choose any name as long as you use only lowercase letters, numbers, hyphens (-), underscores (_) and dots (.))

`gsutil mb gs://qwiklabs-gcp-00-32b1a7b3c0b0`

2. Copy below script and paste it in the Cloud Shell. Before hitting the enter, change the bucket name (In order to set a unique name use your project ID because it is unique. For example, “result_bucket_YOUR_PROJECT_ID” can be your unique bucket name. Or feel free to choose any name as long as you use only lowercase letters, numbers, hyphens (-), underscores (_) and dots (.))

`gsutil mb gs://qwiklabs-gcp-00-32b1a7b3c0b0_2`

3. Copy below script and paste it in the Cloud Shell. Before hitting the enter, change YOUR_TRANSLATE_TOPIC_NAME.

`gcloud pubsub topics create qwiklabs-gcp-00-32b1a7b3c0b0`

4. Copy below script and paste it in the Cloud Shell. Before hitting the enter, change YOUR_RESULT_TOPIC_NAME.

`gcloud pubsub topics create qwiklabs-gcp-00-32b1a7b3c0b0_2`

5. Clone the sample app repository to your Cloud Shell:

`git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git`

6. Change to the directory that contains the Cloud Functions sample code:

`cd python-docs-samples/functions/ocr/app/`

## Deploy the functions

1. To deploy the image processing function with a Cloud Storage trigger, run the following command in the directory that contains the sample code. Replace YOUR_IMAGE_BUCKET_NAME, YOUR_GCP_PROJECT_ID, YOUR_TRANSLATE_TOPIC_NAME and YOUR_RESULT_TOPIC_NAME.

```gcloud functions deploy ocr-extract \
--runtime python39 \
--trigger-bucket qwiklabs-gcp-00-32b1a7b3c0b0 \
--entry-point process_image \
--set-env-vars "^:^GCP_PROJECT=qwiklabs-gcp-00-32b1a7b3c0b0:TRANSLATE_TOPIC=qwiklabs-gcp-00-32b1a7b3c0b0:RESULT_TOPIC=qwiklabs-gcp-00-32b1a7b3c0b0_2:TO_LANG=es,en,fr,ja"```


2. To deploy the text translation function with a Cloud Pub/Sub trigger, run the following command in the directory that contains the sample code. Replace YOUR_TRANSLATE_TOPIC_NAME, YOUR_GCP_PROJECT_ID and YOUR_RESULT_TOPIC_NAME.

```gcloud functions deploy ocr-translate \
--runtime python39 \
--trigger-topic qwiklabs-gcp-00-32b1a7b3c0b0 \
--entry-point translate_text \
--set-env-vars "GCP_PROJECT=qwiklabs-gcp-00-32b1a7b3c0b0,RESULT_TOPIC=qwiklabs-gcp-00-32b1a7b3c0b0_2"```

3. To deploy the function that saves results to Cloud Storage with a Cloud Pub/Sub trigger, run the following command in the directory that contains the sample code. Replace YOUR_RESULT_TOPIC_NAME, YOUR_GCP_PROJECT_ID and YOUR_RESULT_BUCKET_NAME.

```gcloud functions deploy ocr-save \
--runtime python39 \
--trigger-topic qwiklabs-gcp-00-32b1a7b3c0b0_2 \
--entry-point save_result \
--set-env-vars "GCP_PROJECT=qwiklabs-gcp-00-32b1a7b3c0b0,RESULT_BUCKET=qwiklabs-gcp-00-32b1a7b3c0b0_2"```


## Upload an image

1. Download an image from cloud-training bucket.

`gsutil cp gs://cloud-training/OCBL307/menu.jpg .`

2. Upload the image to your Cloud Storage bucket:

`gsutil cp menu.jpg gs://qwiklabs-gcp-00-32b1a7b3c0b0`

3. Watch the logs to be sure the executions have completed:

`gcloud functions logs read --limit 100`

4. You can view the saved translations in the Cloud Storage bucket you used for YOUR_RESULT_BUCKET_NAME.

![Bucket](<Extracting Text from the Images/Storage/Img.png>)

![Text](<Extracting Text from the Images/Storage/Text.png>)

## Delete the Cloud Functions

- Deleting Cloud Functions does not remove any resources stored in Cloud Storage.

To delete the Cloud Functions you created, run the following commands and follow the prompts:

`gcloud functions delete ocr-extract`

`gcloud functions delete ocr-translate`

`gcloud functions delete ocr-save`