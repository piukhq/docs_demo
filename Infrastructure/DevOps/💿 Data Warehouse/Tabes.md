The following is a list of DWH tables which have been created for Bink. Certain tables are PII tables containing PII information. These tables have equivalents within the Bink_secure schema containing the PII columns.

## Fact Tables

### Fact_Transaction

Fact table containing records of transactions. Based of clickhouse events

| EVENT_ID           | PK |                                    |
|--------------------|----|------------------------------------|
| EVENT_DATE_TIME    |    |                                    |
| TRANSACTION_ID     |    | Business PK                        |
| USER_ID            | FK | Joins to user table                |
| PROVIDER_SLUG      | FK | Joins to loyalty card table        |
| LOYALTY_PLAN_NAME  | FK | Joins to loyalty card table        |
| TRANSACTION_DATE   |    |                                    |
| SPEND_AMOUNT       |    |                                    |
| SPEND_CURRENCY     |    |                                    |
| LOYALTY_ID         | FK | Joins to dim loyalty card table    |
| LOYALTY_CARD_ID    | FK | Joins to dim loyalty card table    |
| MERCHANT_ID        | FK | Joins to dim merchant table        |
| PAYMENT_ACCOUNT_ID | FK | Joins to dim payment account table |


### Fact_User

Fact table showing the addition and deletion of users. Based of clickhouse events. PII table available in Bink_Secure

| EVENT_ID           | PK |                                            |
|--------------------|----|--------------------------------------------|
| EVENT_DATE_TIME    |    |                                            |
| USER_ID            | FK | Joins to dim user table                    |
| EVENT_TYPE         |    | Created or deleted                         |
| IS_MOST_RECENT     |    | Flag indicating most recent event per user |
| ORIGIN             |    |                                            |
| CHANNEL            |    |                                            |
| EXTERNAL_USER_REF  |    | PII column                                 |
| EMAIL              |    | PII column                                 |
| DOMAIN             |    |                                            |
| INSERTED_DATE_TIME |    | Auditing column                            |

### Fact_Payment_Account

Fact table showing the addition and removal of payment accounts. Based of clickhouse events. PII table available in Bink_Secure

| EVENT_ID           | PK |                                                       |
|--------------------|----|-------------------------------------------------------|
| EVENT_DATE_TIME    |    |                                                       |
| EVENT_TYPE         |    | added or removed                                      |
| PAYMENT_ACCOUNT_ID | FK | Joins to dim payment account table                    |
| IS_MOST_RECENT     |    | Flag indicating most recent event per payment account |
| ORIGIN             |    |                                                       |
| CHANNEL            |    |                                                       |
| USER_ID            | FK | User connected to dim payment account                 |
| EXTERNAL_USER_REF  |    | PII column                                            |
| EMAIL              |    | PII column                                            |
| EXPIRY_MONTH       |    | PII column                                            |
| EXPIRY_YEAR        |    | PII column                                            |
| EXPIRY_YEAR_MONTH  | FK | Joins to dim date table                               |
| TOKEN              |    |                                                       |
| STATUS             |    |                                                       |
| STATUS_ID          |    |                                                       |
| EMAIL_DOMAIN       |    |                                                       |
| INSERTED_DATE_TIME |    | Auditing column                                       |


### Fact_Payment_Account_Status_Change

Fact table showing the status change of payment accounts. Based of clickhouse events. PII table available in Bink_Secure

| EVENT_ID           | PK |                                                       |
|--------------------|----|-------------------------------------------------------|
| EVENT_DATE_TIME    |    |                                                       |
| EVENT_TYPE         |    | added or removed                                      |
| PAYMENT_ACCOUNT_ID | FK | Joins to dim payment account table                    |
| IS_MOST_RECENT     |    | Flag indicating most recent event per payment account |
| ORIGIN             |    |                                                       |
| CHANNEL            |    |                                                       |
| USER_ID            | FK | User connected to dim payment account                 |
| EXTERNAL_USER_REF  |    | PII column                                            |
| EMAIL              |    | PII column                                            |
| EXPIRY_MONTH       |    | PII column                                            |
| EXPIRY_YEAR        |    | PII column                                            |
| EXPIRY_YEAR_MONTH  | FK | Joins to DIM date table                               |
| TOKEN              |    |                                                       |
| FROM_STATUS_ID     |    |                                                       |
| FROM_STATUS        |    |                                                       |
| TO_STATUS_ID       |    |                                                       |
| TO_STATUS          |    |                                                       |
| EMAIL_DOMAIN       |    |                                                       |
| INSERTED_DATE_TIME |    | Auditing column                                       |

### Fact_Loyalty_Card_*

Set of tables for events pertaining to loyalty card dimension. Add_auth, Auth, Join, Register, Removed. PII table available in Bink_Secure

| EVENT_ID           | PK |                           |
|--------------------|----|---------------------------|
| EVENT_TYPE         |    |                           |
| EVENT_DATE_TIME    |    |                           |
| LOYALTY_CARD_ID    | FK | Joins to Dim Loyalty Card |
| LOYALTY_PLAN       |    |                           |
| IS_MOST_RECENT     |    |                           |
| MAIN_ANSWER        |    | PII column                |
| CHANNEL            |    |                           |
| ORIGIN             |    |                           |
| USER_ID            | FK | Joins to Dim User table   |
| EXTERNAL_USER_ID   |    | PII column                |
| EMAIL              |    | PII column                |
| DOMAIN             |    |                           |
| INSERTED_DATE_TIME |    | Auditing column           |

### Fact_Loyalty_card_Status_Change

Fact table showing the status change of loyalty cards. PII table available in Bink_Secure

| EVENT_ID           | PK |                           |
|--------------------|----|---------------------------|
| EVENT_TYPE         |    |                           |
| EVENT_DATE_TIME    |    |                           |
| LOYALTY_CARD_ID    | FK | Joins to Dim Loyalty Card |
| LOYALTY_PLAN       |    |                           |
| IS_MOST_RECENT     |    |                           |
| MAIN_ANSWER        |    | PII column                |
| FROM_STATUS_ID     |    |                           |
| FROM_STATUS        |    |                           |
| TO_STATUS_ID       |    |                           |
| TO_STATUS          |    |                           |
| CHANNEL            |    |                           |
| ORIGIN             |    |                           |
| USER_ID            | FK | Joins to Dim User table   |
| EXTERNAL_USER_ID   |    | PII column                |
| EMAIL              |    | PII column                |
| DOMAIN             |    |                           |
| INSERTED_DATE_TIME |    | Auditing column           |


## Dimension Tables

### Dim_Channel

| CHANNEL_ID        | PK | Unique PK |
|-------------------|----|-----------|
| CHANNEL_NAME      |    |           |
| ORGANISATION_ID   |    |           |
| ORGANISATION_NAME |    |           |


### Dim_Date

Dim table containing dates from 2010 - 2040.

| DATE          | PK | Unique PK. Timestamp format |
|---------------|----|-----------------------------|
| YEAR          |    | YYYY                        |
| QUARTER       |    |                             |
| MONTH         |    |                             |
| MONTHNAME     |    |                             |
| DAYOFMONTH    |    |                             |
| DAYOFWEEK     |    |                             |
| WEEKOFYEAR    |    |                             |
| DAYOFYEAR     |    |                             |
| DAYNAME       |    |                             |
| WEEKPART      |    | WEEKDAY or WEEKEND          |
| DAYNUMBER     |    |                             |
| YEARNUMBER    |    |                             |
| QUARTERNUMBER |    |                             |
| MONTHNUMBER   |    |                             |
| YEAR_QUARTER  |    | YYYY-Q                      |
| YEAR_MONTH    | FK | YYYY-M. Joins to FACT_USER  |

### Dim_Loyalty_Card

Dim table containing loyalty card records. The loyalty plan dimension is equivalent to scheme. PII table available in Bink_Secure

| LOYALTY_CARD_ID          | PK | Unique PK, Joins to Join loyalty_card_payment_account |
|--------------------------|----|-------------------------------------------------------|
| LINK_DATE                |    |                                                       |
| JOIN_DATE                |    |                                                       |
| CARD_NUMBER              |    | PII column                                            |
| UPDATED                  |    |                                                       |
| STATUS_ID                |    |                                                       |
| STATUS                   |    |                                                       |
| STATUS_TYPE              |    |                                                       |
| STATUS_ROLLUP            |    |                                                       |
| BARCODE                  |    | PII column                                            |
| CREATED                  |    |                                                       |
| ORDERS                   |    |                                                       |
| ORIGINATING_JOURNEY      |    |                                                       |
| IS_DELETED               |    |                                                       |
| LOYALTY_PLAN_ID          | FK |                                                       |
| LOYLATY_PLAN_COMPANY     |    |                                                       |
| LOYALTY_PLAN_SLUG        |    |                                                       |
| LOYALTY_PLAN_TIER        |    |                                                       |
| LOYALTY_PLAN_NAME_CARD   |    |                                                       |
| LOYALTY_PLAN_NAME        |    |                                                       |
| LOYALTY_PLAN_CATEGORY_ID | FK |                                                       |
| LOYALTY_PLAN_CATEGORY    |    |                                                       |


### Dim_Merchant

Dim table containing records for individual merchants.

| MERCHANT_ID                 | PK |                           |
|-----------------------------|----|---------------------------|
| LOCATION                    |    |                           |
| POSTCODE                    |    |                           |
| CREATED_AT                  |    |                           |
| UPDATED_AT                  |    |                           |
| LOCATION_ID                 |    |                           |
| LOYALTY_SCHEME_ID           | FK | Joins to Dim loyalty card |
| LOYALTY_SCHEME_SLUG         |    |                           |
| PAYMENT_PROVIDER_VISA       |    |                           |
| PAYMENT_PROVIDER_MASTERCARD |    |                           |
| PAYMENT_PROVIDER_AMEX       |    |                           |
| MERCHANT_INTERNAL_ID        |    |                           |


### Dim_Payment_Accounts

Dim table containing payment_account and payment card records. PII table available in Bink_Secure.

| PAYMENT_ACCOUNT_ID   | PK | Unique PK, Joins to Join loyalty_card_payment_account |
|----------------------|----|-------------------------------------------------------|
| HASH                 |    | PII column                                            |
| TOKEN                |    | PII column                                            |
| STATUS               |    |                                                       |
| PROVIDER_ID          | FK | Joins to Dim merchant                                 |
| PROVIDER_STATUS_CODE |    |                                                       |
| COUNTRY              |    |                                                       |
| CREATED              |    |                                                       |
| PAN_END              |    |                                                       |
| UPDATED              |    |                                                       |
| CONSENTS_TYPE        |    |                                                       |
| CONSENTS_TIMESTAMP   |    |                                                       |
| CONSENTS_LONGITUDE   |    |                                                       |
| CONSENTS_LATITUDE    |    |                                                       |
| ISSUER_ID            |    |                                                       |
| PAN_START            |    |                                                       |
| PSP_TOKEN            |    | PII column                                            |
| CARD_UID             |    | PII column                                            |
| IS_DELETED           |    |                                                       |
| START_MONTH          |    |                                                       |
| START_YEAR           |    |                                                       |
| EXPIRY_MONTH         |    | PII column                                            |
| EXPIRY_YEAR          |    | PII column                                            |
| FINGERPRINT          |    | PII column                                            |
| ISSUER_NAME          |    |                                                       |
| NAME_ON_CARD         |    | PII column                                            |
| CARD_NICKNAME        |    |                                                       |
| CURRENCY_CODE        |    |                                                       |
| CARD_NAME            |    |                                                       |
| CARD_TYPE            |    |                                                       |


### Dim_User

Dim table containing user records. Contains data about marketing information and social media fields. PII table available in Bink_Secure.

| USER_ID             | PK | Unique PK            |
|---------------------|----|----------------------|
| UID                 |    |                      |
| EXTERNAL_ID         |    |                      |
| CHANNEL_ID          |    | Joins to DIM_CHANNEL |
| DATE_JOINED         |    |                      |
| DELETE_TOKEN        |    |                      |
| EMAIL               |    |                      |
| IS_ACTIVE           |    |                      |
| IS_STAFF            |    |                      |
| IS_SUPERUSER        |    |                      |
| IS_TESTER           |    |                      |
| LAST_LOGIN          |    |                      |
| PASSWORD            |    |                      |
| RESET_TOKEN         |    |                      |
| MARKETING_CODE_ID   |    |                      |
| SALT                |    |                      |
| APPLE               |    |                      |
| FACEBOOK            |    |                      |
| TWITTER             |    |                      |
| MAGIC_LINK_VERIFIED |    |                      |

## Joining tables

### Join_Loyalty_Card_Payment_account

Links loyalty card to payment account and vice-versa. Removes the need for PLLlink IDS and a many to many join.

| PLL_LINK_PK        | PK | Unique PK, effectively a composite primary key |
|--------------------|----|------------------------------------------------|
| LOYALTY_CARD_ID    | FK | Joins to DimLoyaltyCard                        |
| PAYMENT_ACCOUNT_ID | FK | Joins to DimPaymentAccount                     |
| PLL_LINK_ID        |    | Old PLL link ID from source                    |
| ACTIVE_LINK        |    |                                                |
