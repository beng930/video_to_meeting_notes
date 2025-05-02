# 🎥 Video to Meeting Notes – Automated Transcript + Summary Pipeline

This infrastructure automatically transcribes videos and generates structured meeting notes using Amazon Transcribe and Claude 3.5 Sonnet (via Amazon Bedrock).

---

### 💡 What It Does

- You upload **any video file** to a designated folder in S3.
- The system:
  1. Converts the audio to text with **Amazon Transcribe**
  2. Detects and separates different speakers automatically
  3. Sends the transcript to **Claude 3.5 Sonnet** via **Amazon Bedrock**
  4. Generates structured meeting notes
  5. Saves the final notes (in `.txt` format) to a separate S3 folder

---

### ⚙️ Behind the Scenes

- ✅ Upload S3 bucket (`video-upload-bucket-input`)
- ✅ Lambda function (triggered on upload)
- ✅ Amazon Transcribe (turns audio into text)
- ✅ Claude 3.5 (summarizes the text)
- ✅ Output S3 bucket (`video-summary-bucket-outputs`)

![ChatGPT Image May 1, 2025, 08_47_57 PM](https://github.com/user-attachments/assets/1f336463-6075-47a0-8274-01b0fb2dd995)

(No technical actions needed after deployment — just upload a video.)

---

### 🚀 One-Time Setup: How to Deploy

Pull / Download the CFT files to your desired folder on your local machine.
Run the following from your terminal (requires AWS CLI access):

```bash
# Step 1: Deploy the core resources
aws cloudformation deploy \
  --template-file video-processing-core.yaml \
  --stack-name video-processing-core \
  --capabilities CAPABILITY_NAMED_IAM

# Step 2: Get the Lambda ARN
LAMBDA_ARN=$(aws lambda get-function \
  --function-name ProcessVideoAndSummarize \
  --region us-west-2 \
  --query 'Configuration.FunctionArn' \
  --output text)

# Step 3: Attach the Lambda to the upload bucket
aws cloudformation deploy \
  --template-file video-processing-lambda-notifications.yaml \
  --stack-name video-processing-attach \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    UploadBucketName=video-upload-bucket-input \
    LambdaArn=$LAMBDA_ARN \
  --region us-west-2
```

🧠 Note: You need AWS CLI access and the right IAM permissions to deploy the infrastructure.

---

### 📂 How to Use It

1. Upload your video file to the S3 bucket named:  
   `video-upload-bucket-input`

2. Wait a few minutes (processing time depends on video length).

3. Go to the second S3 bucket:  
   `video-summary-bucket-outputs`

4. Download the file ending with `-summary.txt` — this contains your **structured meeting notes**.


The notes include:

- ✅ Attendees (if mentioned in the conversation)
- ✅ Key Topics Discussed
- ✅ Action Items
- ✅ Next Steps

No additional steps are needed. Just upload and receive your notes.

---

### ✏️ Customize the AI Output

To change how the summaries are generated:

1. Open the [AWS Lambda Console](https://console.aws.amazon.com/lambda)
2. Locate the function named `ProcessVideoAndSummarize`
3. Scroll to the source code and find the `prompt_content` definition:

```python
prompt_content = f\"\"\"
As a professional summarizer, create a structured summary of the following conversation transcript...
```
1. Change the tone, structure, or language of the instructions.
2. Click “Deploy” to save.
