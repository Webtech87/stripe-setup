# Setting Up Stripe in Django

1. Install Stripe SDK  
Open your terminal and install the Stripe library:
```bash
pip install stripe
```

2. Configure API Keys  
In `settings.py`, add your Stripe API keys:
```bash
import os
 
STRIPE_SECRET_KEY = os.getenv("STRIPE_SECRET_KEY")  # Backend key 
STRIPE_PUBLISHABLE_KEY = os.getenv("STRIPE_PUBLISHABLE_KEY")  # Frontend key
```
***Note:*** *Get these keys from your Stripe Dashboard under* ***Developers > API keys.***

# Imprementing Stripe Checkout

1. Create a Django View for Checkout Session
In `views.py`, add:
```bash
import stripe 
from django.conf import settings 
from django.http import JsonResponse 
 
def create_checkout_session(request): 
    stripe.api_key = settings.STRIPE_SECRET_KEY 
     
    session = stripe.checkout.Session.create( 
        payment_method_types=["card"], #you can add other payment types
        line_items=[ 
            { 
                "price_data": { 
                    "currency": "usd", 
                    "product_data": {"name": "T-shirt"}, #it can be displayed from database
                    "unit_amount": 2000,  # $20
                }, 
                "quantity": 1, #it can be dynamic from a select input, for example
            } 
        ], 
        mode="payment", 
        success_url="http://localhost:8000/success/", #change domain in production
        cancel_url="http://localhost:8000/cancel/", 
    ) 
    return JsonResponse({"id": session.id})
```  

2. Add URL for Checkout Session  
In `urls.py`, add:  
```bash
from django.urls import path 
from .views import create_checkout_session 
 
urlpatterns = [ 
    path("create-checkout-session/", create_checkout_session, name="checkout"), 
]
```

3. Create Frontend for Checkout Button  

Create `templates/checkout.html` and insert:
```bash
<script src="https://js.stripe.com/v3/"></script> 
<button id="checkout-button">Buy Now</button> 
 
<script> 
  var stripe = Stripe("your_publishable_key_here"); 
  document.getElementById("checkout-button").addEventListener("click", function () { 
      fetch("/create-checkout-session/") 
          .then(response => response.json()) 
          .then(session => { 
              return stripe.redirectToCheckout({ sessionId: session.id }); 
          }) 
          .catch(error => console.error("Error:", error)); 
  }); 
</script>
```

4. Create Success and Cancel Views  
Add in `views.py`:
```bash
def payment_success(request): 
    return HttpResponse("Payment Successful!") 
 
def payment_cancel(request): 
    return HttpResponse("Payment Canceled!")
```