# Implementing Webhooks

Webhooks notify your Django backend when a payment succeeds.  

1. Create Webhook View  
In `views.py`, add: 
```bash
import json 
from django.http import JsonResponse 
 
def stripe_webhook(request): 
    payload = request.body 
    event = None 
 
    try: 
        event = json.loads(payload) 
    except ValueError: 
        return JsonResponse({"error": "Invalid payload"}, status=400) 
 
    if event["type"] == "payment_intent.succeeded": 
        print("Payment successful!") 
 
    return JsonResponse({"status": "success"}, status=200)
```  

2. Add Webhook URL  
In `urls.py`, add: 
```bash
urlpatterns += [ 
    path("stripe-webhook/", stripe_webhook, name="stripe-webhook"), 
]
```  

3. Register Webhook in Stripe  
Go to **Stripe Dashboard > Webhooks** and add:  

http://yourdomain.com/stripe-webhook/ 

4. Test Webhooks Locally  
Use Stripe’s CLI: 
```bash
stripe listen --forward-to localhost:8000/stripe-webhook/
```  
Then trigger a test event: 
```bash
stripe trigger payment_intent.succeeded
```