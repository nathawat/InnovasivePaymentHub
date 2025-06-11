# Payment Integration Module

This document outlines the pseudo-code and expected API interactions needed for a merchant to integrate with **PaymentHUB** for online payments.

## 1. Components Overview

- **Merchant Backend**: Server-side application that communicates with PaymentHUB and manages order logic.
- **Merchant Frontend**: Client-facing interface for customers. Handles redirects and user display.
- **PaymentHUB**: External API (black box) that connects to the payment gateway.
- **Payment Gateway**: External system where the customer authorizes the payment.

## 2. Payment Flow Outline

### 2.1 Payment Initiation & Tokenization

```python
class PaymentService:
    def request_payment_token(self, order_id, amount, currency):
        """Request a token from PaymentHUB for a new transaction."""
        payload = {
            "order_id": order_id,
            "amount": amount,
            "currency": currency
        }
        response = http.post("https://paymenthub.example.com/api/token", json=payload)
        payment_token = response["payment_token"]
        return payment_token
```

### 2.2 Optional: Retrieve Payment Options

```python
    def fetch_payment_options(self, payment_token):
        """Retrieve list of available payment methods."""
        response = http.get(
            f"https://paymenthub.example.com/api/options/{payment_token}"
        )
        options = response["payment_options"]  # e.g., [{"code": "VISA", "name": "Visa"}]
        return options

    def fetch_option_details(self, payment_token, option_code):
        """Get specific fields/logos required for an option."""
        response = http.get(
            f"https://paymenthub.example.com/api/options/{payment_token}/{option_code}"
        )
        return response["payment_option_details"]
```

### 2.3 Payment Execution & Redirection

```python
    def initiate_payment(self, payment_token, amount, currency):
        payload = {
            "payment_token": payment_token,
            "amount": amount,
            "currency": currency
        }
        response = http.post(
            "https://paymenthub.example.com/api/payments", json=payload
        )
        redirect_url = response["redirect_url"]
        return redirect_url
```

- The Merchant Backend returns `redirect_url` to the Merchant Frontend.
- Merchant Frontend redirects the customer's browser to this URL.
- Customer completes authorization on Payment Gateway.

### 2.4 Handling Payment Status & Notifications

```python
    def handle_webhook_callback(self, data):
        """Process backend notification from PaymentHUB."""
        transaction_status = data["transaction_status"]
        order_id = data["order_id"]
        # Update local order state based on transaction_status
        return transaction_status
```

- PaymentHUB sends POST requests to a webhook endpoint on the Merchant Backend with the final `transaction_status`.
- After processing, PaymentHUB redirects the browser back to the Merchant Frontend.

### 2.5 Optional: Payment Inquiry

```python
    def inquire_payment_status(self, payment_token):
        response = http.get(
            f"https://paymenthub.example.com/api/inquiry/{payment_token}"
        )
        return response["payment_inquiry_details"]
```

## 3. Expected API Request/Response Examples

1. **Token Request**

```
POST /api/token
{
    "order_id": "12345",
    "amount": 100.0,
    "currency": "USD"
}

Response
{
    "payment_token": "abc123"
}
```

2. **Initiate Payment**

```
POST /api/payments
{
    "payment_token": "abc123",
    "amount": 100.0,
    "currency": "USD"
}

Response
{
    "redirect_url": "https://gateway.example.com/checkout?token=xyz"
}
```

3. **Webhook Notification**

```
POST /merchant/webhook
{
    "order_id": "12345",
    "transaction_status": "SUCCESS"
}
```

## 4. Redirection and Status Logic

1. **Customer Checkout**: Frontend signals backend to start a payment.
2. **Backend Requests Token**: Uses `request_payment_token` and may optionally fetch payment options.
3. **Initiate Payment**: Backend calls `initiate_payment` and receives `redirect_url`.
4. **Browser Redirect**: Frontend sends user to Payment Gateway using `redirect_url`.
5. **Payment Authorization**: Customer completes payment on the gateway.
6. **Backend Webhook**: PaymentHUB notifies backend via `handle_webhook_callback` with final status.
7. **Frontend Return URL**: PaymentHUB redirects user back to the merchant site where status can be displayed.
8. **Inquiry (Optional)**: If webhook fails, backend can call `inquire_payment_status` to check state.

