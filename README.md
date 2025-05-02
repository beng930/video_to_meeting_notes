Video-to-Meeting-Notes AI Pipeline (Quick Guide)

This tool automatically transcribes and summarizes video recordings into clean, structured meeting notes — all powered by AWS.

💡 What It Does

* You upload any video file to a designated folder in S3.
* The system:

    1. Converts the audio to text with Amazon Transcribe
    2. Detects and separates different speakers automatically
    3. Sends the transcript to Claude 3.5 Sonnet to generate structured meeting notes
    4. Saves the final notes (in .txt format) to another S3 folder for you to download or use.


🛠️ Behind the Scenes

* ✅ Upload S3 bucket (you drop videos here)
* ✅ Lambda function (auto-runs on upload)
* ✅ Amazon Transcribe (turns audio into text) - will be replaced by Nova once CFT support is available
* ✅ Claude 3.5 (summarizes the text)
* ✅ Summary saved to a second S3 bucket

![ChatGPT Image May 1, 2025, 08_47_57 PM](https://github.com/user-attachments/assets/1f336463-6075-47a0-8274-01b0fb2dd995)

(No technical actions needed after deployment — just upload a video.)

📦 One-Time Setup: How to Deploy

You only need to run this once. Here's how to set it up:
```
bash

# Step 1: Deploy core infrastructure
aws cloudformation deploy \
  --template-file video-processing-core.yaml \
  --stack-name video-processing-core \
  --capabilities CAPABILITY_NAMED_IAM
# Step 2: Get Lambda ARN from the output
LAMBDA_ARN=$(aws lambda get-function \
  --function-name ProcessVideoAndSummarize \
  --region us-west-2 \
  --query 'Configuration.FunctionArn' \
  --output text)
# Step 3: Attach upload notification to trigger the Lambda
aws cloudformation deploy \
  --template-file video-processing-lambda-notifications.yaml \
  --stack-name video-processing-attach \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    UploadBucketName=video-upload-bucket-input \
    LambdaArn=$LAMBDA_ARN \
  --region us-west-2
```
Download 
🧠 Note: You need AWS CLI access and the right IAM permissions to deploy.

📂 How to Use It

1. Go to the S3 bucket named:
    video-bucket-for-transcribe-summary
2. Upload your .mp4 file — no special formatting required.
3. Wait 3–5 minutes (depending on length).
4. Go to:
    video-summary-bucket-for-transcribe-summary
     and find your file name ending in -summary.txt.

You’ll get a neat set of meeting notes including:

* Attendees (if mentioned)
* Key topics
* Action items
* Next steps


✏️ Want to Customize the AI Output?

You can easily update the summary style by editing the prompt in the Lambda function:

1. Go to the Lambda Console → ProcessVideoAndSummarize
2. Scroll to the code, look for the section like:

python

prompt_content = f\"\"\"
Summarize the following conversation between speakers...

(you can edit instructions here)
\"\"\"

1. Change the tone, structure, or language of the instructions.
2. Click “Deploy” to save.

Examples:

* Make it more casual
* Focus only on decisions and outcomes
* Get a more lengthy summary

