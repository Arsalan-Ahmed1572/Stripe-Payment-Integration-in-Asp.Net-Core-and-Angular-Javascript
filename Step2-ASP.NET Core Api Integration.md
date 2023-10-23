Create a controller for handling payments:
==========================================
Here, you'll want to implement logic to create a payment intent and return its client secret. The client secret is required on the client-side to complete the payment.

1. Create a model class to handle property from Stripe API

```c#
  public class PaymentModel
        {
            public string Email { get; set; }
            public int UserId { get; set; }
            public string StripeToken { get; set; }
            public decimal Amount { get; set; }
            public int ProjectMappingId { get; set; }
        }
```

2. Create a controller to write API to create or get payment 

=> Create a function that will return ChargeCreateOption Model from input we get , we can also use automapper to map model it will update in next version
```c#
 public ChargeCreateOptions PrepareChargeModel(long amount, string description, string currency, string customerId,string email)
        {
            var model = new ChargeCreateOptions();
            model.Amount = amount * 100;
            model.Description = description;
            model.Currency = currency;
            model.Customer = customerId;
            model.ReceiptEmail = email;
            return model;
        }
```

=> Create a Detail API to create a Stripe Customer ,Charge ,Payment and store some integration key to database

```c# 
        [HttpPost]
        [Route("CreatePayment")]
        public object CreatePayment(PaymentModel paymentModel)
        {
            try
            {
                var user = _userService.GetCustomerById(paymentModel.UserId);
                var charges = new ChargeService();
                var customers = new Stripe.CustomerService();
                string customerId = "";
                var stripeCustomerId = _stripeCustomerMappingService.GetStripeCustomerMappingByCustomerId(user.Id)?.StripeCustomerAccountId;
                if (stripeCustomerId != null)
                {
                    var stripeCustomerExist = customers.Get(stripeCustomerId);
                    if (stripeCustomerExist != null)
                    {
                        customerId = stripeCustomerExist.Id;
                    }
                    else
                    {
                        var customerModel = PrepareCustomerModel(paymentModel.Email, paymentModel.StripeToken, user.Address, user.UserName, user.ContactNumber);
                        var customer = customers.Create(customerModel);
                        var stripeCustomerMappingModel = new StripeCustomerMapping()
                        {
                            CustomerId = user.Id,
                            StripeCustomerAccountId = customer.Id,
                            CreatedOn = DateTime.UtcNow,
                            UpdatedOn = DateTime.UtcNow,
                        };
                        _stripeCustomerMappingService.InsertStripeCustomerMapping(stripeCustomerMappingModel);
                        customerId = customer.Id;
                    }
                }
                else
                {
                    var customerModel = PrepareCustomerModel(paymentModel.Email, paymentModel.StripeToken, user.Address, user.UserName, user.ContactNumber);
                    var customer = customers.Create(customerModel);
                    var stripeCustomerMappingModel = new StripeCustomerMapping()
                    {
                        CustomerId = user.Id,
                        StripeCustomerAccountId = customer.Id,
                        CreatedOn = DateTime.UtcNow,
                        UpdatedOn = DateTime.UtcNow,
                    };
                    _stripeCustomerMappingService.InsertStripeCustomerMapping(stripeCustomerMappingModel);
                    customerId = customer.Id;

                }

                var chargeModel = PrepareChargeModel((long)paymentModel.Amount, "", "USD", customerId, paymentModel.Email);
                var charge = charges.Create(chargeModel);
                if (charge.Status == "succeeded")
                {
                    var ProjectMapping = _customerAgentMappingService.GetById(paymentModel.ProjectMappingId);
                    if (ProjectMapping != null)
                    {
                        ProjectMapping.Status = (int)CustomerProjectMappingStatus.Finished;
                        _customerAgentMappingService.UpdateCustomerAgentMapping(ProjectMapping);
                    }
                    var balanceTrasactionId = charge.BalanceTransactionId;
                    return new { IsSuccess = true, data = balanceTrasactionId, Message = "Payment Created" };
                }
                else
                {
                    return new { IsSuccess = false, Message = charge.Status };
                }
            }
            catch(Exception ex)
            {
                return new { IsSuccess = false, Message = ex.Message };
            }
        }
```
=> Create a API to get Stripe Public Key from backend to integrate save and secure payment
```c#
        [HttpGet]
        [Route("GetStripePublicableKey")]
        public object GetStripePublicableKey()
        {
            var result = _config.GetValue<string>("Stripe:PublishableKey");
            return new { IsSuccess = true, Data =result };
        }
```

