# Google Sheets Integration Setup Guide

This guide will help you set up Google Sheets to automatically save form submissions from your landing page.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet
3. Name it "Maili Beta Access Requests" (or any name you prefer)
4. In the first row, add these column headers:
   - **A1:** Name
   - **B1:** Email
   - **C1:** Status
   - **D1:** Timestamp
   - **E1:** Request Date
   - **F1:** IP Address

## Step 2: Create Google Apps Script

1. In your Google Sheet, click **Extensions** → **Apps Script**
2. Delete any existing code and paste this script:

```javascript
function doPost(e) {
  try {
    // Parse the incoming data
    const data = JSON.parse(e.postData.contents);
    
    // Get the active spreadsheet
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Append the new row
    sheet.appendRow([
      data.name || 'Not provided',
      data.email || '',
      data.status || 'Pending',
      data.timestamp || new Date().toISOString(),
      data.request_date || new Date().toLocaleString(),
      data.ip_address || 'Unknown'
    ]);
    
    // Return success response
    return ContentService.createTextOutput(JSON.stringify({
      result: 'success',
      message: 'Data saved successfully'
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    // Return error response
    return ContentService.createTextOutput(JSON.stringify({
      result: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

// Optional: Test function to verify setup
function test() {
  const testData = {
    name: 'Test User',
    email: 'test@example.com',
    status: 'Pending',
    timestamp: new Date().toISOString(),
    request_date: new Date().toLocaleString(),
    ip_address: '127.0.0.1'
  };
  
  const mockEvent = {
    postData: {
      contents: JSON.stringify(testData)
    }
  };
  
  const result = doPost(mockEvent);
  Logger.log(result.getContent());
}
```

## Step 3: Deploy as Web App

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type" and choose **Web app**
3. Configure the deployment:
   - **Description:** Maili Beta Access Form Handler
   - **Execute as:** Me (your email)
   - **Who has access:** Anyone (important for public access)
4. Click **Deploy**
5. **Copy the Web App URL** - you'll need this for the next step
6. Click **Authorize access** and grant permissions when prompted

## Step 4: Update Your Landing Page

1. Open `landing-page/index.html`
2. Find this line in the JavaScript section:
   ```javascript
   const GOOGLE_SHEETS_WEB_APP_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
   ```
3. Replace `YOUR_GOOGLE_APPS_SCRIPT_URL_HERE` with your Web App URL from Step 3
4. Save the file

## Step 5: Test the Integration

1. Open your landing page
2. Fill out the "Request Access" form
3. Submit the form
4. Check your Google Sheet - you should see a new row with the submitted data
5. Check your email - you should receive the EmailJS notification

## Troubleshooting

### Data not appearing in Sheets?
- Make sure you set "Who has access" to "Anyone" when deploying
- Check the Apps Script execution log: **Executions** tab in Apps Script editor
- Verify the Web App URL is correct in your HTML file

### Permission errors?
- Make sure you authorized the script when deploying
- Re-deploy and authorize again if needed

### CORS errors?
- The code uses `mode: 'no-cors'` which is normal for Google Apps Script
- You won't see the response, but data should still be saved

## Optional: Format Your Sheet

You can format the sheet to make it look better:

1. **Freeze the header row:** Select row 1, then **View** → **Freeze** → **1 row**
2. **Format as table:** Select all data, then **Format** → **Alternating colors**
3. **Auto-resize columns:** Select all columns, then double-click between column headers
4. **Add filters:** Select row 1, then **Data** → **Create a filter**

## Security Note

The Web App URL is public, but:
- It only accepts POST requests with specific data structure
- You can add validation in the Apps Script if needed
- Consider adding rate limiting for production use

## Viewing Your Data

Your Google Sheet will automatically update with each form submission. You can:
- View it in Google Sheets
- Export to Excel: **File** → **Download** → **Microsoft Excel**
- Share with team members
- Set up email notifications for new rows (via Apps Script triggers)

