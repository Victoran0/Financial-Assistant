# General Workflow

Creating account with phone number and email (optional) and the account have a password
upon creation. Take the user to a page where they add more details to their account (transaction pin, payment details etc)

`Hi {{user_name}}, how may i assist you financially?`

## The Agent Workflow

Authentication is required to access agent, else simply provide a simple llm with only 5 response per day, (limited help)

Authenticated user

1. get the user's prompt
2. CATEGORIZE_INITIAL_PROMPT (categories: PURCHASE_REQUEST, FINANCIAL_ASSISTANT (greeting, finance, other questions etc), TRANSACTION_HISTORY, PLATFORM_NAVIGATION (issues navigating the platform/finding a specific section in the app), CUSTOMER_SUPPORT_REQUIRED). return the category

3. use the CATEGORIZE_INITIAL_PROMPT RESPONSE + the initial prompt to process the request

   - PURCHASE_REQUEST (the user's prompts should be categorized before feeding back to the agent):

     - VALIDATE_PAYMENT_METHOD if the user have not linked their payment details (backend api call), POPUP the payment details page
       - if they provide the details, route to PURCHASE_INFORMATION_VALIDATION
       - if they do not provide the details or they close the popup, END
     - PURCHASE_INFORMATION_VALIDATION check the user's request if it contains every detail required to process the purchase
       - If it contains every details required to make the purchase, navigate to GET_PURCHASE_DATA
       - if it does not contain every required details, ask the user for the missing details: PROVIDE_MISSING_PURCHASE_INFORMATION
         - PROVIDE_MISSING_PURCHASE_INFORMATION: check if they provided the missing details and recategorize as (categories: PROVIDED_MISSING_PURCHASE_INFORMATION, OTHER_QUERIES).
           - if they PROVIDED_MISSING_PURCHASE_INFORMATION call GET_PURCHASE_DATA
           - If OTHER_QUERIES, navigate back to CATEGORIZE_INITIAL_PROMPT
       - GET_PURCHASE_DATA: if it contains every required details, return the required details as a json
         - VALIDATE_TRANSACTION_PIN: show popup (human-in-the-loop) that asks for their transaction pin, if they dont have one yet, tell them to create it and ask for their password to verify its the user creating the pin.
           - if the pin/password is wrong, return false and send message that the password/pin is incorrect, else return true and process the purchase for the user. Once done, append the transaction to their transaction history, save to vector db and let agent send a response that the transaction is done successfully. END
           - if they do not provide the details or they close the popup, END

   - FINANCIAL_ASSISTANT:

     - GET_FINANCIAL_ASSISTANT_RESPONSE: A good Prompt of how llm should reply and restrictions (the chat must be finance related, Asides greetings and general topics): financial, savings and investment related.
     - run CATEGORIZE_INITIAL_PROMPT on every new message

   - TRANSACTION_HISTORY:

     - If the user is asking about one of their transactions, run backend api call on the vector db and give them the response
     - run CATEGORIZE_INITIAL_PROMPT on every new message

   - PLATFORM_NAVIGATION:

     - If the user is having issues navigating the platform/finding a specific section in the app, check the vector db of the platform navigations and give them a response
     - run CATEGORIZE_INITIAL_PROMPT on every new message

   - CUSTOMER_SUPPORT_REQUIRED:
     - Human-in-the-loop to step in and help them resolve their advanced, expert required problem (we can provide a simple email to contact for a start)

### Agents

1. CATEGORIZE_INITIAL_PROMPT
2. PURCHASE_REQUEST
3. FINANCIAL_ASSISTANT
4. TRANSACTION_HISTORY
5. CUSTOMER_SUPPORT_REQUIRED
6. PLATFORM_NAVIGATION

7. VALIDATE_PAYMENT_METHOD
8. PURCHASE_INFORMATION_VALIDATION
9. PROVIDE_MISSING_PURCHASE_INFORMATION
10. PROVIDED_MISSING_PURCHASE_INFORMATION
11. OTHER_QUERIES
12. GET_PURCHASE_DATA
13. VALIDATE_TRANSACTION_PIN

14. GET_FINANCIAL_ASSISTANT_RESPONSE
15. END
