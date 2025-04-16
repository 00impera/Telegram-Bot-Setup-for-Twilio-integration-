# Setting Up a Telegram Bot for Twilio Integration

## Step 1: Create a Telegram Bot using BotFather

1. Open Telegram and search for "BotFather" or click this link: https://t.me/botfather
2. Start a conversation with BotFather by clicking "Start" or typing "/start"
3. Type `/newbot` to create a new bot
4. Follow the prompts:
   - Enter a display name for your bot (e.g., "TwilioTelegramBot")
   - Enter a username for your bot (must end with 'bot', e.g., "Twilio_telegram_bot")
5. BotFather will provide you with a token - **save this token securely** as we'll need it for the integration

## Step 2: Get Your Chat ID

1. Start a conversation with your newly created bot by sending any message
2. To get your chat ID, visit this URL in your browser:
   ```
   https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
   ```
   (Replace `<YOUR_BOT_TOKEN>` with the token provided by BotFather)

3. Look for the `"chat":{"id":XXXXXXXXX}` field in the JSON response
4. This number is your chat ID (it will be a positive number for personal chats or a negative number for group chats)

## Step 3: Create PHP Webhook to Forward Twilio Messages to Telegram

Now let's create a PHP script that will receive messages from Twilio and forward them to your Telegram bot:

```php
<?php
// webhook.php - Forwards Twilio SMS messages to Telegram

// Telegram configuration
$telegramBotToken = 'YOUR_BOT_TOKEN'; // Replace with your bot token
$chatId = 'YOUR_CHAT_ID';             // Replace with your chat ID

// Get incoming message details from Twilio
$messageFrom = $_POST['From'] ?? 'Unknown';
$messageBody = $_POST['Body'] ?? 'No message content';

// Format message for Telegram
$telegramMessage = "New SMS from: $messageFrom\n\nMessage: $messageBody";

// Send message to Telegram
$telegramApiUrl = "https://api.telegram.org/bot$telegramBotToken/sendMessage";
$telegramData = [
    'chat_id' => $chatId,
    'text' => $telegramMessage,
    'parse_mode' => 'HTML'
];

// Initialize cURL session
$ch = curl_init($telegramApiUrl);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $telegramData);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$telegramResponse = curl_exec($ch);
curl_close($ch);

// Log the response (optional)
file_put_contents('telegram_log.txt', 
    date('Y-m-d H:i:s') . " - Telegram API Response: $telegramResponse\n", 
    FILE_APPEND);

// Send TwiML response to Twilio
header("content-type: text/xml");
echo '<?xml version="1.0" encoding="UTF-8"?><Response></Response>';
?>
```

## Step 4: Deploy and Configure

1. Save the script above as `webhook.php` in your local server directory
2. Replace `YOUR_BOT_TOKEN` and `YOUR_CHAT_ID` with your actual values
3. Ensure your PHP server is running: `php -S localhost:3000`
4. Create an ngrok tunnel: `./ngrok http 3000`
5. Configure your Twilio number to use the ngrok URL (e.g., `https://your-ngrok-url.ngrok.io/webhook.php`) as the webhook for incoming messages

## Testing the Integration

1. Send an SMS to your Twilio phone number
2. If everything is set up correctly, you should receive the message via your Telegram bot

## Troubleshooting

If messages aren't being forwarded:
- Check the `telegram_log.txt` file for any error messages
- Verify your bot token and chat ID are correct
- Make sure your ngrok tunnel is active and the URL is properly configured in Twilio
- Check that your PHP server is running<?php
// webhook.php - Forwards Twilio SMS messages to Telegram

// Telegram configuration
$telegramBotToken = 'YOUR_BOT_TOKEN'; // Replace with your bot token from BotFather
$chatId = 'YOUR_CHAT_ID';             // Replace with your chat ID

// Get incoming message details from Twilio
$messageFrom = $_POST['From'] ?? 'Unknown';
$messageBody = $_POST['Body'] ?? 'No message content';
$messageTo = $_POST['To'] ?? 'Unknown';
$messageMedia = isset($_POST['NumMedia']) && $_POST['NumMedia'] > 0 ? true : false;

// Format message for Telegram
$telegramMessage = "📱 *New SMS Message*\n\n";
$telegramMessage .= "From: `$messageFrom`\n";
$telegramMessage .= "To: `$messageTo`\n";
$telegramMessage .= "Message: \n\n$messageBody";

if ($messageMedia) {
    $mediaCount = $_POST['NumMedia'];
    $telegramMessage .= "\n\n📎 *Has $mediaCount media attachment(s)*";
    // Note: Forwarding media attachments would require additional code
    // to download and upload the files to Telegram
}

// Send message to Telegram
$telegramApiUrl = "https://api.telegram.org/bot$telegramBotToken/sendMessage";
$telegramData = [
    'chat_id' => $chatId,
    'text' => $telegramMessage,
    'parse_mode' => 'Markdown'
];

// Initialize cURL session
$ch = curl_init($telegramApiUrl);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $telegramData);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$telegramResponse = curl_exec($ch);
$curlError = curl_error($ch);
curl_close($ch);

// Log the transaction
$logMessage = date('Y-m-d H:i:s') . " - From: $messageFrom, To: $messageTo\n";
$logMessage .= "Message: $messageBody\n";
$logMessage .= "Telegram API Response: $telegramResponse\n";
if ($curlError) {
    $logMessage .= "cURL Error: $curlError\n";
}
$logMessage .= "----------------------------\n";
file_put_contents('twilio_telegram_log.txt', $logMessage, FILE_APPEND);

// Send TwiML response to Twilio
header("content-type: text/xml");
echo '<?xml version="1.0" encoding="UTF-8"?><Response></Response>';
?>
