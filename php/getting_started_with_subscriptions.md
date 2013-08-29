Getting started with subscriptions
==================================

There is just two steps to have a fully functional subscription system in place.

1. Customers can pay for subscriptions
2. Retrieve callbacks to automate subscriptions

1. Customers can pay for subscriptions
--------------------------------------

```php
// Set the secret API key
\Peakium.::$api_key = "you_secret_api_key"

// Is this request a create new or update existing subscription?
$update_request = false;

// Parameters to build the submission form
$params = array(
	'customer' => 'some_customer_id',
	'subscription' => 'some_subscription_id',
	'item_id' => 101,
	'item_description' => 'My new subscription',
	'period_amount' => 1,
	'period_type' => 'month',
	'payment_amount' => 500,
	'payment_currency' => 'USD',
	'payment_discount' => 0,
	'return_url_ok' => 'http://example.com/page_customer_should_see_on_success/',
	'return_url_error' => 'http://example.com/page_customer_should_see_on_error/',
	'submit_button_text' => 'Continue to payment',
);

// The following code will print full HTML so it is a good idea to stop any output after this
$submission_form = \Peakium\SubmissionForm::build(($update_request ? 'update-subscription' : 'create-subscription'), $params);
print $submission_form['html'];
```

2. Retrieve callbacks to automate subscriptions
-----------------------------------------------

```php
$fp = fopen('php://input', 'r');
$raw_data = stream_get_contents($fp);
$event = json_decode($raw_data, true);

switch($event['event']):
	// In case an invoice has been paid
	case 'invoice.paid':
		$customer = $event['data']['customer']['id'];
		$payment_date = $event['data']['timestamp_paid'];
		foreach($event['data']['items'] as $item)
		{
			// Is the item a subscription?
			if (!isset($item['subscription']))
				continue;

			$subscription = $item['subscription']['id'];
			$period_end = $item['subscription']['period_end'];
			$amount = $item['total_amount'];
			$currency = $item['currency'];

			// Update subscription expiration
		}
	break;

	// In case an invoice could not be paid
	case 'invoice.payment_failed':
		$customer = $event['data']['customer']['id'];
		$due_date = $event['data']['due'];
		$amount_to_be_paid = $event['data']['calculated_total']['amount'];
		$currency = $event['data']['calculated_total']['currency'];

		// Notify customer or you
	break;

	default:
		throw new Exception(sprintf('Could not handle event "%s"', $event['event']));
endswitch;
```

2b. Using return path
---------------------

An alternative is to use the invoice id when the customer returns to your website. It is not as reliable, because users might not return to your website for some reason, so use this method with care!

```php
// Set the secret API key
\Peakium.::$api_key = "you_secret_api_key"

$retrieve_params = array('expand' => array('item.subscription'));
$invoice = \Peakium\Invoice::retrieve($_GET['invoice'], $retrieve_params);
if ($invoice['paid'] != true)
	throw new Exception('Invoice has not been paid.');

$customer = $event['data']['customer']['id'];
$payment_date = $event['data']['timestamp_paid'];
foreach($invoice['items'] as $item)
{
	// Is the item a subscription?
	if (!isset($item['subscription']))
		continue;

	$subscription = $item['subscription']['id'];
	$period_end = $item['subscription']['period_end'];
	$amount = $item['total_amount'];
	$currency = $item['currency'];

	// Update subscription expiration
}
```