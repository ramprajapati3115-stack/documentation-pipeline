# documentation-pipeline

An n8n workflow for extracting text from document images, classifying common document types, and routing low-confidence results for human review.

## Workflow

The workflow is stored in the repository-root file [`json`](./json). It:

1. Accepts a document image at a webhook and validates that an image was provided.
2. Hashes the image and checks MongoDB for a previously processed document.
3. Uses Google Cloud Vision OCR to extract text.
4. Classifies the text as an invoice, ID card, certificate, or unknown and calculates a confidence score.
5. Retries low-confidence OCR with alternate language hints, up to three attempts.
6. Sends low-confidence results to Slack for manual review and stores the result in MongoDB.

## Requirements

- An n8n instance with Webhook, Code, HTTP Request, MongoDB, IF, Wait, and Respond to Webhook nodes available.
- A Google Cloud Vision API key with the Vision API enabled.
- A MongoDB database and collection for `extracted_documents`.
- A Slack incoming webhook for human-review notifications.

Configure credentials and service endpoints in n8n before activating the workflow. The workflow file contains example MongoDB credential references and a placeholder Slack webhook URL; replace or map these to credentials configured in your own n8n instance. Do not commit API keys, database credentials, or real webhook URLs.

## Import and configure

1. In n8n, import the workflow from the repository-root `json` file.
2. Configure the MongoDB credentials and database/collection used by the deduplication and storage nodes.
3. Configure Google Vision authentication for the OCR request.
4. Replace the Slack webhook placeholder with a securely configured endpoint.
5. Test the workflow, then activate it and copy the webhook URL.

## Webhook request

Send a `POST` request to the active webhook with a JSON body containing the image as a base64-encoded string. `docId` is optional; when omitted, the workflow generates one.

```json
{
  "image": "<base64-encoded-image>",
  "docId": "optional-document-id"
}
```

The workflow returns a JSON response with a status such as `processed`, `duplicate`, `escalated`, or `error`. Processed results include the document type, confidence, and attempt count.

## Repository contents

- `json` — n8n workflow definition.
- `README.md` — setup and usage instructions.
- `.gitignore` — excludes local secrets and generated files.

## Security

Keep credentials in n8n's credential store or another approved secret manager. Keep local `.env` files and generated document data out of Git. The repository does not include a standalone Python application or a `requirements.txt`; run the workflow in n8n.
