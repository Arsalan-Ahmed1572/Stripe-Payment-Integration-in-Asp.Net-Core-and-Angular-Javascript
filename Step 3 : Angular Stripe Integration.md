Client-Side (Angular):
To integrate a Stripe payment modal in an Angular 8 application, you can use Stripe's client-side JavaScript library, Stripe.js. Here's a step-by-step guide on how to add a Stripe payment modal to your Angular 8 project:

Step 1: Set Up Your Angular 8 Project

If you haven't already, create an Angular 8 project. You can use Angular CLI for this:
```bash
ng new your-angular-project
cd your-angular-project
ng serve
```

Step 2: Install Stripe.js

In your Angular project, you need to install Stripe.js to handle payments and create a payment modal. You can do this by adding the Stripe.js library to your project's index.html file:

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- ... other head elements ... -->
    <script src="https://js.stripe.com/v3/"></script>
  </head>
  <!-- ... rest of your HTML ... -->
</html>
```

Step 3: Create a Payment Component

Create an Angular component that will handle the payment process. You can use the Angular CLI to generate a new component:
```bash
ng generate component payment
```

Step 4: Set Up Stripe Configuration in Your Component

In your payment.component.ts, you need to configure Stripe with your public key:

```typescript
import { Component, OnInit } from '@angular/core';

declare var Stripe: any;
declare var stripeTocken:any;
@Component({
  selector: 'app-payment',
  templateUrl: './payment.component.html',
  styleUrls: ['./payment.component.css']
})
export class PaymentComponent implements OnInit {
  stripe: any;
  elements: any;

  constructor() {
    // get the key from controller api
    this.stripe = GetStripePublicableKey();
    
  }

  ngOnInit(): void {
    // Initialize Stripe.js here   
       loadStripe();
}

2.	loadStripe() {
3.	       
4.	      if(!window.document.getElementById('stripe-script')) {
5.	        var s = window.document.createElement("script");
6.	        s.id = "stripe-script";
7.	        s.type = "text/javascript";
8.	        s.src = "https://checkout.stripe.com/checkout.js";
9.	        s.onload = () => {
10.	          this.handler = (<any>window).StripeCheckout.configure({
11.	            key:this.stripeKey,
12.	            locale: 'auto',
13.	            token: function (token: any) {
14.	              
15.	              // You can access the token ID with `token.id`.
16.	              // Get the token ID to your server-side code for use.
19.	            }
20.	          });
21.	        }
22.	         
23.	        window.document.body.appendChild(s);
24.	      }
25.	    }



}
```
2. Button click function to show stripe payment modal with amount 

```typescript
pay(amount: any ,id : any) {    
    var $this = this;
   
      this.stripeTocken = "";
      var handler = (<any>window).StripeCheckout.configure({
        
        key:this.stripeKey, //'pk_test_51JkURuJY2HE8zI3EDegHPwlSXDbztggnorpBGejJTwJV1xw4aOwLhdspUmN63zowO4suwtXGiA2EcB7UUQQNNflr00zfNL1KDb',
        locale: 'auto',
        token: function (token: any) {
          this.stripeTocken = token.id;
          $this.sendrequest(this.stripeTocken,amount,id);
        }
      });
      handler.open({
                name: ' Payment',
                description: 'Project Configuration',
                amount: amount *100
              });
    }
```

3. After Enter information of card details click on pay,Send request is used for sending stripe tocken amount to the controller

```typescript
sendrequest(tockenId :any,amount :any,id : any)
  {
  this.blockUI.start('Loading...');
  var model = {
    StripeToken : tockenId,
    UserId : this.User.id,
    Email : this.User.email,
    Amount : parseFloat(amount),
    ProjectMappingId :parseInt(id)
  }
  this.configuratorService.createPayment(model).subscribe(res=>{
              if(res.isSuccess){
                this.blockUI.stop();
                Swal.fire({
                  title: 'Project Configuration!',
                  text: 'Payment Submitted',
                  imageUrl: '../../../../../assets/images/backgrounds/stripe-icon-3.jpg',
                  imageWidth: 400,
                  imageHeight: 200,
                  imageAlt: 'Custom image',
                }).then((result) => {
                  this.router.navigate(['/app/projectConfiguration']);
              
              })
              }
              else
              {
                this.loader = false;
                Swal.fire({
                  position: 'top-end',
                  icon: 'error',
                  title: 'Payment Error',
                  text: res.message,
                  showConfirmButton: true,
                })
              }
            })
  }
```

=> All the code in steps are merged in single TS file it is divided here because for demostration
