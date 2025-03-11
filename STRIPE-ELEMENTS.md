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

# Implementing Stripe Elements

1. Create a Payment Intent View  
In `views.py`, add:  
```bash
def create_payment_intent(request): 
    stripe.api_key = settings.STRIPE_SECRET_KEY 
    intent = stripe.PaymentIntent.create( 
        amount=2000,  # $20 
        currency="usd", 
        payment_method_types=["card"], 
    ) 
    return JsonResponse({"clientSecret": intent.client_secret})
```  

2. Add URL for Payment Intent  
in `urls.py`, add:  
```bash
urlpatterns += [ 
    path("create-payment-intent/", create_payment_intent, name="payment-intent"), 
]
```  

3. Create Payment Form for Elements  
Create `templates/payment.html` and insert:  
```bash
<script src="https://js.stripe.com/v3/"></script> 
<div id="card-element"></div> 
<button id="pay-button">Pay</button> 
 
<script> 
  var stripe = Stripe("your_publishable_key_here"); 
  var elements = stripe.elements(); 
  var card = elements.create("card"); 
  card.mount("#card-element"); 
 
  document.getElementById("pay-button").addEventListener("click", function () { 
      fetch("/create-payment-intent/") 
          .then(response => response.json()) 
          .then(data => { 
              stripe.confirmCardPayment(data.clientSecret, { 
                  payment_method: { card: card } 
              }).then(result => { 
                  if (result.error) { 
                      console.log(result.error.message); 
                  } else { 
                      console.log("Payment Successful!"); 
                  } 
              }); 
          }); 
  }); 
</script>
```