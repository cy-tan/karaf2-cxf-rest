The REST service provides a customer service that supports the following operations

- PUT /customerservice/customers/ - to create or update a customer
- GET /customerservice/customers/{id} - to view a customer with the given id
- DELETE /customerservice/customers/{id} - to delete a customer with the given id
- GET /customerservice/orders/{orderId} - to view an order with the given id
- GET /customerservice/orders/{orderId}/products/{productId} - to view a specific product on an order with the given id

* Retrieve the customer instance with id 123
      curl http://quickstart-cxf-rest.f8/cxf/crm/customerservice/customers/123
