import fs from 'fs';

// إنشاء ملف main.js
const mainJsContent = `
import axios from 'axios';
import * as cheerio from 'cheerio';
import { Actor } from 'apify';

await Actor.init();

try {
    const input = {
        url: "https://www.talabat.com/jordan/groceries",
        apiToken: "apify_api_KbTubUPQab5UsBUmBnxiajD1pYijj12vddgf",
        targetApiUrl: "https://your-api-endpoint.com/receive-data"
    };

    const { url, apiToken, targetApiUrl } = input;

    if (!url || !apiToken || !targetApiUrl) {
        throw new Error('Missing required inputs: url, apiToken, or targetApiUrl.');
    }

    console.log(\`Fetching data from: \${url}\`);

    const response = await axios.get(url);
    const $ = cheerio.load(response.data);

    const stores = [];
    $(".store-card").each((i, element) => {
        stores.push({
            name: $(element).find(".store-name").text().trim(),
            category: $(element).find(".store-category").text().trim(),
            deliveryTime: $(element).find(".delivery-time").text().trim(),
            rating: $(element).find(".rating").text().trim()
        });
    });

    console.log('Extracted stores:', stores);

    const apiResponse = await axios.post(targetApiUrl, {
        data: stores,
        sourceUrl: url,
    }, {
        headers: {
            Authorization: \`Bearer \${apiToken}\`,
            'Content-Type': 'application/json',
        },
    });

    console.log('Data successfully sent to the target API:', apiResponse.data);

    await Actor.pushData(stores);
} catch (error) {
    console.error('Error:', error.message);
} finally {
    await Actor.exit();
}
`;
fs.writeFileSync('main.js', mainJsContent);

// إنشاء ملف package.json
const packageJsonContent = `
{
  "name": "talabat-scraper",
  "version": "1.0.0",
  "type": "module",
  "main": "main.js",
  "scripts": {
    "start": "node main.js"
  },
  "dependencies": {
    "axios": "^1.5.0",
    "cheerio": "^1.0.0-rc.12",
    "apify": "^3.2.6"
  }
}
`;
fs.writeFileSync('package.json', packageJsonContent);

// إنشاء ملف Dockerfile
const dockerfileContent = `
FROM apify/actor-node:20

COPY package*.json ./
RUN npm install --omit=dev --omit=optional
COPY . ./

CMD npm start --silent
`;
fs.writeFileSync('Dockerfile', dockerfileContent);

// إنشاء ملف input_schema.json (اختياري)
const inputSchemaContent = `
{
  "title": "Input Schema",
  "type": "object",
  "properties": {
    "url": {
      "type": "string",
      "description": "URL to scrape",
      "default": "https://www.talabat.com/jordan/groceries"
    },
    "apiToken": {
      "type": "string",
      "description": "API token",
      "default": "apify_api_KbTubUPQab5UsBUmBnxiajD1pYijj12vddgf"
    },
    "targetApiUrl": {
      "type": "string",
      "description": "Target API URL",
      "default": "https://your-api-endpoint.com/receive-data"
    }
  },
  "required": ["url", "apiToken", "targetApiUrl"]
}
`;
fs.writeFileSync('.actor/input_schema.json', inputSchemaContent);

console.log('All necessary files have been created successfully.');