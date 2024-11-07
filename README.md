# WATI.IO

To set up WATI integration with OpenAI in a Laravel application, you’ll need to create routes, controllers, and helper functions to manage webhook requests, process responses with OpenAI, and send replies back through WATI. Here’s a step-by-step guide for setting this up in Laravel.

### Step 1: Set Up Routes

In your `web.php` or `api.php` routes file, define a route to handle incoming webhook requests from WATI:

```php
use App\Http\Controllers\WatiController;

Route::post('/wati/webhook', [WatiController::class, 'handleWebhook']);
```

### Step 2: Create WatiController

In the `app/Http/Controllers` directory, create `WatiController.php` to handle the webhook and communication with OpenAI.

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class WatiController extends Controller
{
    public function handleWebhook(Request $request)
    {
        $incomingMessage = $request->input('message');
        $userNumber = $request->input('from');

        // Get AI response from OpenAI
        $aiResponse = $this->getAIResponse($incomingMessage);

        // Send response back to user through WATI
        $this->sendMessageToWATI($userNumber, $aiResponse);

        return response()->json(['status' => 'success'], 200);
    }

    private function getAIResponse($message)
    {
        $response = Http::withHeaders([
            'Authorization' => 'Bearer ' . env('OPENAI_API_KEY'),
        ])->post('https://api.openai.com/v1/completions', [
            'model' => 'gpt-4',
            'prompt' => $message,
            'max_tokens' => 100,
        ]);

        return trim($response->json()['choices'][0]['text']);
    }

    private function sendMessageToWATI($number, $message)
    {
        return Http::withHeaders([
            'Authorization' => 'Bearer ' . env('WATI_API_KEY'),
        ])->post('https://api.wati.io/v1/sendMessage', [
            'to' => $number,
            'message' => $message,
        ]);
    }
}
```

### Step 3: Configure API Keys

In your `.env` file, add the following variables to store your API keys securely:

```env
OPENAI_API_KEY=your_openai_api_key
WATI_API_KEY=your_wati_api_key
```

### Step 4: Test the Integration

1. **Set up the Webhook in WATI**: Log into your WATI dashboard, navigate to **Settings > Webhooks**, and set the webhook URL to your Laravel endpoint (e.g., `https://yourdomain.com/wati/webhook`).
2. **Send a Message**: Message your WhatsApp Business number and see if the response is automatically generated and sent through OpenAI and WATI.

### Additional Tips

- **Error Handling**: Add error handling to manage cases where requests to OpenAI or WATI fail.
- **Logging**: You may want to log responses for debugging or improve response quality.

This setup will allow your Laravel app to process incoming messages through OpenAI and WATI, automating responses on WhatsApp! Let me know if you’d like further customization or help with testing.
