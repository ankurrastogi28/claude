---
type: reference
tags:
  - subscription
  - catalog
  - stripe
  - sandbox
  - definitions
  - buc-483
captured: 2026-09-04
account: AnkurSandbox acct_1RE6OuCu7Eq6LYq9 (livemode false)
---
# Subscription plan definitions — sandbox (Pro / Pro Scale)

Captured 2026-09-04 for the Subscription Package Catalog work ([[subscription-package-catalog-design]] / BUC-483). This is the corrected definitions set, and as of capture the **sandbox Stripe catalog was updated to match it exactly** (all `usage-limit` + `next-level` metadata wired, `pro_annual_5/10` renamed to `_50/_100`, `pro_annual_500` fixed 1000→500, `proscale_annual_500` next-level de-self-referenced). Verified against a fresh Stripe read.

**Account:** AnkurSandbox `acct_1RE6OuCu7Eq6LYq9` (livemode false). **Products:** Pro `prod_V0lX3PjdMUov68` (2 seats), Pro Scale `prod_V0lXeqYdOsXhhC` (5 seats), Free `prod_V0lXj5OvQGbv6q` (no price). Prices GBP ex-VAT. Ladder per interval per package: **50 → 100 → 500 → 1000 → 2000**. `level_uuid7` is assigned at seed (per environment — sandbox ids ≠ prod ids). NOTE: these `stripe_price_id`s are **sandbox** ids; prod will differ.

```csv
package,level_lookup_key,nickname,interval,band_usage_limit,price_gbp,currency,stripe_product_id,stripe_price_id,next_level_price_id,level_uuid7
Parcelhero Pro,pro_monthly_50,Pro Monthly — up to 50/mo,month,50,23.75,gbp,prod_V0lX3PjdMUov68,price_1U0k0ACu7Eq6LYq9NWX6IlvM,price_1U0k0ACu7Eq6LYq960iMXxca,(assigned at seed)
Parcelhero Pro,pro_monthly_100,Pro Monthly — up to 100/mo,month,100,36.25,gbp,prod_V0lX3PjdMUov68,price_1U0k0ACu7Eq6LYq960iMXxca,price_1U0k0BCu7Eq6LYq9pkSqi18D,(assigned at seed)
Parcelhero Pro,pro_monthly_500,Pro Monthly — up to 500/mo,month,500,48.75,gbp,prod_V0lX3PjdMUov68,price_1U0k0BCu7Eq6LYq9pkSqi18D,price_1U0k0CCu7Eq6LYq9XMrpLXBl,(assigned at seed)
Parcelhero Pro,pro_monthly_1000,Pro Monthly — up to 1000/mo,month,1000,86.25,gbp,prod_V0lX3PjdMUov68,price_1U0k0CCu7Eq6LYq9XMrpLXBl,price_1U0k0CCu7Eq6LYq9CNaAroCd,(assigned at seed)
Parcelhero Pro,pro_monthly_2000,Pro Monthly — up to 2000/mo,month,2000,123.75,gbp,prod_V0lX3PjdMUov68,price_1U0k0CCu7Eq6LYq9CNaAroCd,,(assigned at seed)
Parcelhero Pro,pro_trial_zero_monthly,£0 free PH Pro monthly trial,month,,0,gbp,prod_V0lX3PjdMUov68,price_1U0k0HCu7Eq6LYq9ZrKZbGtG,,(assigned at seed)
Parcelhero Pro,pro_annual_50,Pro Annual — up to 50/mo,year,50,228,gbp,prod_V0lX3PjdMUov68,price_1U0k0DCu7Eq6LYq9EsJvzDnO,price_1U0k0ECu7Eq6LYq98jgWhaNO,(assigned at seed)
Parcelhero Pro,pro_annual_100,Pro Annual — up to 100/mo,year,100,348,gbp,prod_V0lX3PjdMUov68,price_1U0k0ECu7Eq6LYq98jgWhaNO,price_1U0k0FCu7Eq6LYq988pIJDX9,(assigned at seed)
Parcelhero Pro,pro_annual_500,Pro Annual — up to 500/mo,year,500,468,gbp,prod_V0lX3PjdMUov68,price_1U0k0FCu7Eq6LYq988pIJDX9,price_1U0k0FCu7Eq6LYq9AH2knniT,(assigned at seed)
Parcelhero Pro,pro_annual_1000,Pro Annual — up to 1000/mo,year,1000,828,gbp,prod_V0lX3PjdMUov68,price_1U0k0FCu7Eq6LYq9AH2knniT,price_1U0k0GCu7Eq6LYq9EuMXJMHh,(assigned at seed)
Parcelhero Pro,pro_annual_2000,Pro Annual — up to 2000/mo,year,2000,1188,gbp,prod_V0lX3PjdMUov68,price_1U0k0GCu7Eq6LYq9EuMXJMHh,,(assigned at seed)
Parcelhero Pro Scale,proscale_monthly_50,Pro Scale Monthly — up to 50/mo,month,50,61.25,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0HCu7Eq6LYq9XeDTCcg6,price_1U0k0ICu7Eq6LYq9xCOu9QLi,(assigned at seed)
Parcelhero Pro Scale,proscale_monthly_100,Pro Scale Monthly — up to 100/mo,month,100,86.25,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0ICu7Eq6LYq9xCOu9QLi,price_1U0k0JCu7Eq6LYq9rMr0i1FT,(assigned at seed)
Parcelhero Pro Scale,proscale_monthly_500,Pro Scale Monthly — up to 500/mo,month,500,123.75,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0JCu7Eq6LYq9rMr0i1FT,price_1U0k0KCu7Eq6LYq99Ip4XxiA,(assigned at seed)
Parcelhero Pro Scale,proscale_monthly_1000,Pro Scale Monthly — up to 1000/mo,month,1000,186.25,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0KCu7Eq6LYq99Ip4XxiA,price_1U0k0KCu7Eq6LYq90Sr6jVwi,(assigned at seed)
Parcelhero Pro Scale,proscale_monthly_2000,Pro Scale Monthly — up to 2000/mo,month,2000,248.75,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0KCu7Eq6LYq90Sr6jVwi,,(assigned at seed)
Parcelhero Pro Scale,proscale_trial_zero_monthly,£0 free PH Pro Scale monthly trial,month,,0,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0OCu7Eq6LYq9bdgewZv9,,(assigned at seed)
Parcelhero Pro Scale,proscale_annual_50,Pro Scale Annual — up to 50/mo,year,50,588,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0LCu7Eq6LYq9lXjM1jQu,price_1U0k0MCu7Eq6LYq9fdjxHXH5,(assigned at seed)
Parcelhero Pro Scale,proscale_annual_100,Pro Scale Annual — up to 100/mo,year,100,828,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0MCu7Eq6LYq9fdjxHXH5,price_1U0k0MCu7Eq6LYq9IOmL4IWg,(assigned at seed)
Parcelhero Pro Scale,proscale_annual_500,Pro Scale Annual — up to 500/mo,year,500,1188,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0MCu7Eq6LYq9IOmL4IWg,price_1U0k0NCu7Eq6LYq9jytg2o6U,(assigned at seed)
Parcelhero Pro Scale,proscale_annual_1000,Pro Scale Annual — up to 1000/mo,year,1000,1788,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0NCu7Eq6LYq9jytg2o6U,price_1U0k0OCu7Eq6LYq9a1NcS1La,(assigned at seed)
Parcelhero Pro Scale,proscale_annual_2000,Pro Scale Annual — up to 2000/mo,year,2000,2388,gbp,prod_V0lXeqYdOsXhhC,price_1U0k0OCu7Eq6LYq9a1NcS1La,,(assigned at seed)
Parcelhero Free,,Parcelhero Free (default tier),,,,,prod_V0lXj5OvQGbv6q,,,(assigned at seed)
```

Local copy: `Pistacia/.tmp/subscription-definitions-sandbox-draft.csv`. When Ticket 3 seeds, it persists these + assigns the uuid7 ids (per environment).
