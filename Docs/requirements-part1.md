## 1- Problem summary

PayLink is a company in Egypt. Millions of people use it to pay utility bills (electricity, water, mobile top-up, internet) from a digital wallet, instead of  queuing at the biller's office.

PayLink holds customers' money and depends on outside billers it does not control.
Things can go wrong at any step: the app may send the same payment twice, a biller may answer too late, or two payments may leave the same wallet at the same moment. Each biller also works in a different way (fast, slow, delayed, or by daily file).

When this happens, a customer may be charged twice or for a payment the biller never received.
PayLink then loses money and trust, and may face regulators.
Management needs a new backend that handles these cases safely and can easily add new billers.


## 2- Stakeholders

1- Customers: people who pay their bills from a wallet; they need to be charged the right amount, and never twice for the same payment.
2- Agents: small shops that pay bills for customers; they need to earn their commission.
3- Billers: companies that issue bills (ElectroCo, AquaCo, TopUpNet, NetCo); they need to be paid fast and correctly.
4- Admin: PayLink staff who manage the system; they need to change billers, fees and limits easily.
5- Support employees: PayLink staff who help customers; they need to find any payment and block accounts.
6- Management: PayLink leaders; they need profit, customer trust and a system that is easy to grow.
7- Regulators: government bodies that watch the company; they need proof that no money is lost.
8- SMS/Push providers: outside services that send messages; they need to receive requests correctly, and PayLink must keep working if they are down.


## 3- Actors

Users
1- Customer: pays bills from the wallet and manages their account.
2- Agent: pays bills for customers and earns a commission.
3- Admin: manages billers, fees and limits.
4- Support Employee: finds payments and blocks accounts.

System actors
1-Biller System: receives payments from PayLink (ElectroCo, AquaCo, TopUpNet, NetCo).
2-SMS/Push Provider: sends notifications to customers.
3-Scheduler: runs nightly reconciliation and payment retries.

## 4- User Stories

US-1 [FR-1]: Register Account
As a customer, I want to register an account, so that I can use a wallet.
Given I am a new user, when I register with valid details, then an account is created with a basic verification level and a wallet with 0.00 EGP.

US-2 [FR-2]: Upgrade to Verified
As an admin, I want to upgrade a customer to verified, so that their limits are raised.
Given a basic customer with a 5,000 EGP daily limit, when the admin upgrades them to verified, then their daily limit becomes 30,000 EGP.

US-3 [FR-3]: View Wallet and History
As a customer, I want to see my available balance, held balance and transaction history, so that I know what I can spend.
Given I have 500.00 EGP available and 352.50 EGP held, when I open my wallet, then I see both balances and a list of my transactions.

US-4 [FR-4]: Top Up Wallet
As a customer, I want to top up my wallet by simulated card or agent cash, so that I can pay bills.
Given my available balance is 100.00 EGP, when I top up 200.00 EGP by simulated card, then my available balance becomes 300.00 EGP.

US-5 [FR-5]: Record Ledger Entries
As the system, I want every balance change to create a ledger entry, so that balances are computed from the ledger and can be audited.
Given any top-up, hold, charge or refund, when it happens, then a ledger entry is created and my balance equals the sum of my entries.

US-6 [FR-6]: Manage Billers
As an admin, I want to add, edit and disable billers and services, so that the catalog stays current.
Given an active biller, when the admin disables it, then customers can no longer see it or pay it.

US-7 [FR-7]: View Billers and Fields
As a customer, I want to see the billers and the input fields each requires, so that I know what to enter.
Given ElectroCo is active, when I open its service, then I see that a meter number is required.

US-8 [FR-8]: Run Bill Inquiry
As a customer, I want to run a bill inquiry and see the amount, fee and total, so that I know the cost before paying.
Given a meter with a 350.00 EGP bill, when I run an inquiry, then I see amount 350.00, fee 2.50 and total 352.50 EGP.

US-9 [FR-9]: Pay a Bill
As a customer, I want to pay a bill from my wallet, so that I avoid queuing.
Given my available balance is 500.00 EGP and the total is 352.50 EGP, when I confirm the payment, then 352.50 EGP is held, the biller is called, and I receive a receipt when it completes.
Given my available balance is 100.00 EGP and the total is 352.50 EGP, when I confirm the payment, then it is Rejected because of insufficient funds.

US-10 [FR-10]: Use Idempotency Key
As a customer, I want a repeated request with the same idempotency key to return the original result, so that I am never charged twice.
Given I already paid with key K1, when the app sends the same request with K1 again, then no second charge is made and the first result is returned.

US-11 [FR-11]: Enforce Payment Life Cycle
As the system, I want every payment to follow only the allowed life cycle transitions, so that its state is always correct.
Given a payment in Completed state, when any event tries to move it to another state, then the transition is refused and the state stays Completed.
Given a payment is Unknown, when the biller later confirms, then it becomes Completed; and when the biller later refuses, then it becomes Failed.

US-12 [FR-12]: Hold and Release Funds
As a customer, I want my money held before the biller is called and released if the payment fails, so that I never lose money.
Given a payment of 352.50 EGP is held, when the biller refuses it, then the payment becomes Failed, the held amount is released to my available balance, and the payment becomes Refunded.

US-13 [FR-13]: Re-query Unknown Payments
As a customer, I want a timed-out payment to be re-queried, so that it is resolved without me paying twice.
Given a payment is Unknown after the 10-second timeout, when the retry rules run, then it is re-queried after 1, 5 and 15 minutes, and it is never refunded automatically.

US-14 [FR-14]: Process TopUpNet Callback
As the system, I want to accept asynchronous callbacks from TopUpNet, so that mobile top-ups are completed or failed correctly.
Given a top-up payment was accepted by TopUpNet, when TopUpNet calls back with "success", then the payment becomes Completed.
Given a top-up payment was accepted by TopUpNet, when TopUpNet calls back with "failed", then the payment becomes Failed and the held     amount is released to my wallet.

US-15 [FR-15]: Support Multiple Billers
As a customer, I want to pay billers with different formats (ElectroCo, AquaCo, TopUpNet), so that I can pay all my bills in one place.
Given active electricity, water and mobile top-up services, when I pay each one, then every payment is sent in that biller's own format and processed correctly.

US-16 [FR-16]: Calculate Service Fee
As a customer, I want the correct service fee applied, so that I am charged fairly.
Given a 500.00 EGP water bill, when I run an inquiry, then the fee is 2.50 EGP (fixed for electricity and water).
Given a 200.00 EGP mobile top-up, when I run an inquiry, then the fee is 3.00 EGP (1.5%, minimum 1 EGP).
Given a 150.00 EGP internet bill, when I run an inquiry, then the fee is 2.00 EGP.
Given a 200.00 EGP internet bill, when I run an inquiry, then the fee is 2.00 EGP.
Given a 300.00 EGP internet bill, when I run an inquiry, then the fee is 4.00 EGP.
Given an active promotion on a biller, when I run an inquiry for it, then the fee is 0.00 EGP.

US-17 [FR-17]: Pay as Agent
As an agent, I want to pay a bill on behalf of a customer and earn a commission, so that I am rewarded for my service.
Given a payment with a 2.50 EGP fee made at my terminal, when it completes, then 1.00 EGP (40% of the fee) is credited to my wallet.

US-18 [FR-18]: Enforce Payment Limits
As a customer, I want payments above the limits rejected with a clear reason, so that I understand why.
Given I am a basic customer who already paid 4,900.00 EGP today, when I try to pay 200.00 EGP, then the payment is Rejected with the reason "daily limit exceeded".
Given I am a verified customer who already paid 29,900.00 EGP today, when I try to pay 200.00 EGP, then the payment is Rejected with the reason "daily limit exceeded".
Given a single payment of 4.00 EGP, when I submit it, then it is Rejected because it is below the 5 EGP minimum.
Given a single payment of 10,500.00 EGP, when I submit it, then it is Rejected because it is above the 10,000 EGP maximum.

US-19 [FR-19]: Block on Velocity
As the system, I want to block an account that makes too many payments quickly, so that fraud is limited.
Given I made 5 payments in the last 60 seconds, when I make a 6th, then it is rejected and my account is blocked for 10 minutes.

US-20 [FR-20]: Search Payment by Reference
As a support employee, I want to search a payment by reference and see its full history, so that I can help customers.
Given a payment with reference R1, when I search R1, then I see every state change, with times, from Initiated to its current state.

US-21 [FR-21]: Block and Unblock Account
As a support employee, I want to block and unblock an account, so that I can stop suspicious activity.
Given an active account, when I block it, then any new payment from it is Rejected until I unblock it.

US-22 [FR-22]: Send Payment Notifications
As a customer, I want an SMS and a push message on completion and on failure, so that I know the result.
Given my 352.50 EGP electricity payment completes, when it becomes Completed, then I receive an SMS and a push message confirming the payment.
Given my payment fails because the biller refused it, when it becomes Failed, then I receive an SMS and a push message that state the reason.

US-23 [FR-23]: Retry Failed Notifications
As the system, I want failed notifications retried 3 times and then marked undelivered, so that messages are not lost silently.
Given an SMS fails to send, when the system retries, then it tries 3 times and then marks the message undelivered.

US-24 [FR-24]: Edit Fees and Limits
As an admin, I want to edit fee and limit values without redeploying, so that rules can change quickly.
Given the basic daily limit is 5,000 EGP, when the admin changes it to 6,000 EGP, then the new limit applies to the next payment without a redeploy.

US-25 [FR-25]: Write Audit Log
As the system, I want every money movement and admin action written to an audit log, so that everything can be traced.
Given any money movement or admin action, when it happens, then an audit record is saved with who, when and why.

US-26 [FR-26]: Run Daily Reconciliation
As a support employee, I want a daily reconciliation report from the settlement file, so that differences are found.
Given a settlement file, when the Scheduler runs the nightly job, then PayLink's payments are compared with the file and a difference report is produced.

US-27 [FR-27]: Request Refund
As a customer, I want to request a refund of a completed payment, so that I can recover money from a mistake.
Given a Completed payment, when I request a refund and support approves it, then the amount is returned to my wallet with a ledger entry.
Given a Completed payment, when I request a refund and support rejects it, then the payment stays Completed and I am told the reason.
Given support approved a refund, when crediting the wallet fails, then the refund stays pending, no money is lost, and the failure is logged for retry.

US-28 [FR-28]: View Daily Totals
As an admin, I want to view daily totals, so that I can monitor the business.
Given a day with payments, when I open the dashboard, then I see total payments, fees earned, commissions paid and failures per biller.

## Hard-Case Stories

US-29 [HC-1]: Duplicate Payment from Slow Network
As a customer, I want a repeated payment request not to charge me twice, so that one payment means one charge.
Given the app sent the same payment twice because the network was slow, when both requests reach the server, then there is one payment and one charge, and the second request returns the first result.

US-30 [HC-2]: ElectroCo Timeout
As a customer, I want to be told my payment is being processed when ElectroCo is slow, so that I am not confused.
Given ElectroCo answers after 12 seconds, when the 10-second timeout passes, then the payment becomes Unknown, I am told it is being processed, and my money stays held.

US-31 [HC-3]: AquaCo Refusal
As a customer, I want my money back and a clear reason when AquaCo refuses my payment, so that I know what happened.
Given AquaCo refuses with an error code, when the refusal arrives, then the payment becomes Failed, the hold is released to my wallet, and I receive an SMS with a clear reason.

US-32 [HC-4]: Concurrent Payments from One Wallet
As the system, I want simultaneous payments from one wallet handled safely, so that the balance never goes negative.
Given 400.00 EGP available and two payments of 352.50 EGP arrive at the same moment, when both are processed, then one succeeds, the other is Rejected, and the balance is never negative.

US-33 [HC-5]: SMS Provider Down
As the system, I want a payment to complete even when the SMS provider is down, so that notification problems never block money.
Given the SMS provider is down, when a payment completes, then the payment stays Completed, the SMS is retried 3 times, and it is then marked undelivered.

US-34 [HC-6]: Duplicate TopUpNet Callback
As the system, I want a second "success" callback from TopUpNet ignored, so that a payment completes only once.
Given a payment already Completed by a "success" callback, when TopUpNet sends "success" again, then the second callback is ignored and logged.

US-35 [HC-7]: Settlement Difference
As a support employee, I want a payment that the settlement file lists but PayLink has as Failed to be flagged, so that I can review it.
Given the nightly settlement file lists a payment PayLink has as Failed, when reconciliation runs, then the difference is flagged in the reconciliation report for support to review.

US-36 [HC-8]: Add a New Biller
As the PayLink team, I want to add a new biller by adding only an integration and configuration, so that existing modules stay unchanged.
Given a new biller must be added, when the team adds it, then only a new integration and configuration are added and no existing module changes.








