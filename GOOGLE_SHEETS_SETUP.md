# Flight Tracker: Google Sheets Setup

Follow these steps to connect the flight form on the site to a Google Sheet.

## Step 1: Create a Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet
2. Name it something like "PC Bachelor Party - Flights"
3. In row 1, add these headers (exactly as written):

| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| Name | Airline | Flight Number | Arrival | Departure | Submitted | Type |

4. Leave everything else blank

## Step 2: Add the Apps Script

1. In your Google Sheet, click **Extensions > Apps Script**
2. Delete any code in the editor
3. Paste the following code:

```javascript
function doGet(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = sheet.getDataRange().getValues();
  var headers = data[0];
  var rows = [];
  for (var i = 1; i < data.length; i++) {
    var row = {};
    for (var j = 0; j < headers.length; j++) {
      var val = data[i][j];
      if (val instanceof Date) {
        row[headers[j]] = val.toISOString();
      } else {
        row[headers[j]] = val;
      }
    }
    rows.push(row);
  }
  return ContentService.createTextOutput(JSON.stringify(rows))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var params = JSON.parse(e.postData.contents);
  sheet.appendRow([
    params.name || '',
    params.airline || '',
    params.flightNumber || '',
    params.arrival || '',
    params.departure || '',
    new Date().toISOString(),
    params.type || 'arrival'
  ]);
  return ContentService.createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. Click the save icon (or Ctrl+S) and name the project "Flight Tracker"

## Step 3: Deploy as a Web App

1. Click **Deploy > New deployment**
2. Click the gear icon next to "Select type" and choose **Web app**
3. Set these options:
   - **Description**: Flight Tracker
   - **Execute as**: Me
   - **Who has access**: Anyone
4. Click **Deploy**
5. Click **Authorize access**, then choose your Google account
6. If you see a "Google hasn't verified this app" warning, click **Advanced > Go to Flight Tracker (unsafe)**. This is safe; it is your own script.
7. Copy the **Web app URL** (it looks like `https://script.google.com/macros/s/ABC.../exec`)

## Step 4: Add the URL to the Site

Give the URL to me (in Cursor) and I will plug it into the site code. Or, if editing manually, find this line in `index.html`:

```javascript
const SHEETS_API_URL = '';
```

And paste your URL between the quotes.

## That's it!

Once the URL is set, the flight form and board on the site will be live. Anyone with the site password can submit and view flight info.
