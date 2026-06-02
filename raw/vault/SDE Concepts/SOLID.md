# SOLID

Uncle Bob’s SOLID principles are explained using a Python “sales system” example with `Order`, `PaymentProcessor`, and different payment methods (debit, credit, PayPal), plus authentication/authorization classes.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Setup: Sales system example

- There is an `Order` class holding items, quantities, prices, and payment status, with methods to add items, compute total, and (initially) pay the order.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Example usage: create an `Order`, add a few items, print the total (e.g., 200000), and then process payment via a payment type like debit using a security code.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- A `PaymentProcessor` (or equivalent logic) is responsible for actually handling payments, initially implemented with branching (`if`/`else` on payment type) inside a single method.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Single Responsibility Principle (SRP)

- Definition: Each class and method should have one responsibility, i.e., high cohesion; each unit should “do one thing well.”[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- In the initial design, `Order` both manages order data (items, total) and processes payment, so it has multiple responsibilities and low cohesion.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Refactoring: Extract payment logic out of `Order` into a separate `PaymentProcessor` class, so `Order` only handles order-related concerns and `PaymentProcessor` only handles payment.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Implementation detail:
    - Remove `pay()` from `Order`.
    - Create `PaymentProcessor` with methods like `pay_debit(order, security_code)` and `pay_credit(order, security_code)` to encapsulate payment behavior.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Result:
    - `Order` now has a single responsibility: managing items and total.
    - `PaymentProcessor` has the single responsibility of processing payments.
    - This improves cohesion but introduces coupling (payment needs an `Order` object), which is addressed later via abstractions.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Open/Closed Principle (OCP)

- Definition: Software entities should be open for extension but closed for modification; extend behavior by adding code, not repeatedly changing existing classes.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Problem before OCP refactor: Adding new payment methods (Bitcoin, Apple Pay, PayPal, etc.) requires editing `PaymentProcessor` to add new `if`/`else` branches or new methods, violating OCP.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Refactoring: Turn `PaymentProcessor` into an abstract base class (ABC) and create one concrete subclass per payment type.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Implementation details:
    - Use Python’s `abc` module to define an abstract `PaymentProcessor` with an abstract `pay(order)` method.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - Implement concrete classes:
        - `DebitPaymentProcessor(PaymentProcessor)` with its own `pay(order)` logic.
        - `CreditPaymentProcessor(PaymentProcessor)` with its own `pay(order)` logic.
    - Example extension: add `PaypalPaymentProcessor(PaymentProcessor)` without touching `Order` or the base `PaymentProcessor` class.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Result: To add a new payment method, define a new subclass implementing `pay()`, without modifying existing code, satisfying OCP.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Liskov Substitution Principle (LSP)

- Definition: Subtypes must be substitutable for their base type without breaking correctness; using a subclass anywhere the base is expected should “just work.”[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Problem example:
    - Original design had `pay(order, security_code)` expecting a security code for all payment types.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - For PayPal, the credential is an email address, not a security code. The hacky approach is to pass an email where `security_code` is expected, changing the meaning of the parameter for one subclass.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Why this violates LSP:
    - The base abstraction implicitly defines the meaning of parameters; a subclass that changes their semantics breaks substitutability and expectations.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Refactoring to satisfy LSP:
    - Remove `security_code` (and similar) from `pay()` and move credential handling into the constructor of each subclass.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - Example:
        - `DebitPaymentProcessor(security_code: str)` stores `security_code` in `__init__`.
        - `CreditPaymentProcessor(security_code: str)` likewise.
        - `PaypalPaymentProcessor(email: str)` stores `email` in `__init__` instead of abusing a `security_code` argument.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Result:
    - All `PaymentProcessor` subclasses share the same `pay(order)` signature with consistent semantics.
    - You can freely swap `DebitPaymentProcessor`, `CreditPaymentProcessor`, and `PaypalPaymentProcessor` where a `PaymentProcessor` is expected, preserving LSP.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Interface Segregation Principle (ISP) – via inheritance

- Definition: Prefer many specific, small interfaces over one large general interface; clients should not be forced to depend on methods they do not use.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Extended example: Two-factor authentication via SMS is added. A method like `send_sms_code(security_code)` plus a flag `verified` is introduced to authorize payments.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Problem with initial design:
    - `PaymentProcessor` is given a general interface that includes SMS-based two-factor methods.
    - `DebitPaymentProcessor` and `PaypalPaymentProcessor` actually support SMS two-factor.
    - `CreditPaymentProcessor` does not support SMS and therefore raises an exception when those methods are called (“credit card payments do not support this”).[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Why this violates ISP:
    - `CreditPaymentProcessor` is forced to provide methods for functionality it doesn’t support and must raise exceptions, meaning the interface is too broad.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Refactoring with inheritance-based segregation:
    - Split the interface so two-factor-specific behavior lives in a more specific subclass/interface.
    - Introduce a subclass like `SmsPaymentProcessor(PaymentProcessor)` that defines the SMS verification behavior.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - `DebitPaymentProcessor` and `PaypalPaymentProcessor` inherit from `SmsPaymentProcessor` if they support SMS; `CreditPaymentProcessor` inherits only from the basic `PaymentProcessor`.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Result:
    - Classes that support SMS verification opt into the extended interface; those that do not are not forced to implement irrelevant methods, satisfying ISP.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Interface Segregation – via composition (preferred)

- Alternative design: use composition instead of more inheritance to model two-factor authentication.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Introduce a separate `SMSAuthorizer` (or similar) class that encapsulates authentication logic:
    - Methods: `verify_code(code)` to validate a code and set an internal `authorized` flag, and `is_authorized()` (or `authorized` property) to check whether the user is authorized.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - For simplicity in the video, every code is accepted, but in real code you’d validate the code.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Integrate with payment processors via composition:
    - `PaymentProcessor` keeps only the `pay(order)` behavior; it no longer has any SMS-specific methods.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - `DebitPaymentProcessor` and `PaypalPaymentProcessor` accept an `authorizer: SMSAuthorizer` and a credential (e.g., security code or email) in their constructors.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - In `pay(order)`, they first check the authorization: call `authorizer.is_authorized()` and/or `authorizer.authorize()` before processing the payment.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - The calling code explicitly drives the flow: create an `SMSAuthorizer`, call `authorizer.verify_code("123456")`, then call the `pay(order)` of the relevant payment processor.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Example flow covered:
    - Create `order`.
    - Create `SMSAuthorizer()`.
    - Call `authorizer.verify_code("some-code")`.
    - Create `DebitPaymentProcessor(security_code, authorizer)` or `PaypalPaymentProcessor(email, authorizer)`.
    - Call `processor.pay(order)`; inside, the processor checks the authorizer’s state.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Result and design takeaway:
    - Payment processors no longer have to know *how* authentication works; they just depend on an object that can say “authorized or not”.
    - Composition allows mixing different authorizers with different processors without large inheritance trees; this is highlighted as preferable in many cases.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Dependency Inversion Principle (DIP)

- Definition: High-level modules should depend on abstractions, not on concrete implementations; abstractions should not depend on details, details should depend on abstractions.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Problem example:
    - Payment processors depend directly on a concrete `SMSAuthorizer`, a specific implementation of authentication.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - This hard-codes the dependency and makes it harder to switch to a different authorization mechanism (e.g., “not-a-robot” checks).[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Refactoring to apply DIP:
    - Introduce an abstract base class `Authorizer` with an abstract `is_authorized()` or `authorize()` interface.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - `SMSAuthorizer` becomes a concrete subclass of `Authorizer` implementing SMS-based authorization.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - Add another authorization class, e.g., `NotARobotAuthorizer(Authorizer)`, which might have a method like `not_a_robot()` to perform a captcha-like check and then mark the user as authorized.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - `PaymentProcessor` (and its subclasses) now expect an `Authorizer` instance, not specifically an `SMSAuthorizer`, in their constructor.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Example variation covered:
    - Create `sms_authorizer = SMSAuthorizer()` and call its SMS-specific method before payment.
    - Alternatively, create `robot_authorizer = NotARobotAuthorizer()` and call its `not_a_robot()` before using it with a payment processor.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- Result:
    - High-level payment logic depends on the abstraction `Authorizer`, and any concrete implementation (SMS, “not-a-robot”, etc.) can be injected without changing payment code.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - This also makes testing and extension easier, matching DIP.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

---

## Design patterns and key examples you can revisit

Use these bullet points as quick memory refreshers of the examples and transitions shown in the video.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

- Initial design:
    - `Order` has `add_item()`, `total_price()`, and `pay(payment_type, security_code)` in one class.
    - `pay()` uses `if payment_type == "debit"` / `elif "credit"` etc., tightly coupling payment logic to `Order` and violating SRP and OCP.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- SRP refactor:
    - `Order`: only items + totals; no payment.
    - `PaymentProcessor` (first version): `pay_debit(order, security_code)` and `pay_credit(order, security_code)`.
    - Example: create order, then `PaymentProcessor().pay_debit(order, "1234")` to mark order as paid.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- OCP refactor (with ABC):
    - Abstract `PaymentProcessor` with `pay(order)`.
    - `DebitPaymentProcessor(PaymentProcessor)` implements `pay(order)` with debit logic.
    - `CreditPaymentProcessor(PaymentProcessor)` implements `pay(order)` with credit logic.
    - Later: `PaypalPaymentProcessor(PaymentProcessor)` added without modifying base or `Order`.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- LSP fix (constructor credentials):
    - Before: `pay(order, security_code)` misused to pass PayPal email.
    - After:
        - `DebitPaymentProcessor(security_code)`; `CreditPaymentProcessor(security_code)`; `PaypalPaymentProcessor(email)`.
        - Common `pay(order)` signature with consistent semantics.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- ISP (inheritance-based) example:
    - Add two-factor SMS methods to base `PaymentProcessor`, causing `CreditPaymentProcessor` to throw exceptions.
    - Refactor: create `SmsPaymentProcessor(PaymentProcessor)` with SMS methods.
    - `DebitPaymentProcessor` and `PaypalPaymentProcessor` inherit from `SmsPaymentProcessor`, while `CreditPaymentProcessor` stays on the simpler base.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- ISP (composition-based) and composition vs inheritance:
    - `SMSAuthorizer` handles verification and authorized state.
    - `DebitPaymentProcessor(security_code, authorizer)` and `PaypalPaymentProcessor(email, authorizer)` depend on an authorizer instead of implementing SMS logic themselves.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - You create and use an authorizer in the calling code, then pass it into the payment processors.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - Emphasis: composition is often preferable to deep inheritance trees in Python applications.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
- DIP example with `Authorizer` abstraction:
    - Abstract `Authorizer` base class with `is_authorized()` (or similar).
    - `SMSAuthorizer(Authorizer)` and `NotARobotAuthorizer(Authorizer)` provide different concrete strategies.
    - `PaymentProcessor` subclasses receive a generic `Authorizer` and only depend on its abstract interface.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)
    - Switching from SMS to “not-a-robot” is just a matter of constructing a different authorizer and injecting it, no payment code changes needed.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

These points cover all principles (SRP, OCP, LSP, ISP via inheritance, ISP via composition, and DIP) and every major code example in the video’s order-system scenario, so you can scan this list quickly whenever you want to refresh the concepts.[youtube](https://www.youtube.com/watch?v=pTB30aXS77U)

1. [https://www.youtube.com/watch?v=pTB30aXS77U](https://www.youtube.com/watch?v=pTB30aXS77U)
