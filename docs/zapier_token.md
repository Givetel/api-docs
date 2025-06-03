---
title: Givetel Authentication Using Zapier
---

# /token Zap Configuration

This is a Sub-Zap, which is a Zap that can be called by another Zap. This is useful when you need a utility that can be used across many Zaps if needed, which is the case for authentication endpoints like `/token`.

The following are the steps involved in setting up the `/token` Sub-Zap. Each heading will represent a new step in the Zap.

## 1. Create a Trigger - Sub-Zap by Zapier

- Select the Sub-Zap by Zapier app
- Click continue through until you can run a test and get a test record. This record will be blank and we won't do anything with it.

## 2. Get Token & Expiry If Exists

- Select the Storage by Zapier app for this step. You may be prompted to set this app up for your account in this step.
- Name this step "Get Token & Expiry From Storage" or similar.
- Choose the Action Event `Get Multiple Value` and press Next.
- In the `Configure` tab, enter the keys `token` and `token_expires`.
- Note that this step won't succeed when you test it yet, because we haven't stored a token - this happens later.

## 3. Get The Current Time in UTC

- Select the `Code` Zapier app for this step.
- Name this step "Get Current Timestamp" or similar.
- Select the action event `Run Javascript`.
- Use the following code in the `Configure` tab:

```javascript
return { now: new Date().toISOString() };
```

- Test the step and test that the current time in UTC is being returned.

## 4. Check if Local Token is Expired

- Select the `Formatter` Zapier app for this step.
- Name this step "Has Token Expired?" or similar.
- Select the action event `Date/Time`.
- In the `Configure` tab, set the `Start Date` to the current date & time value from the previous step and set the `End Date` to the value from step 2. For the `Date Format - Start Date` you want to use `YYYY-MM-DDTHH:mm:ssZ` (note the T between the DD and the HH), and for the `Date Format - End Date` you want to use `YYYY-MM-DD HH:mm:ss Z` (note the space beteen the DD and the HH).

## 5. Split Into Paths

- Select the `Paths` Zapier app for this step.

## 6. Set Path Conditions for Refresh Token

- In the left-hand side of the path you've created, click `Path Conditions` and set the type of your rules to `Custom Rules`.
- In the `Only continue if` section, select the `Token` value from step 2 and select `Does not exist`.
- Click `Add "Or" rule group`, and configure the new condition to cntinue if the `Output Dates Swapped` value from step 4 is true.

### 7. Get New Token

- Select the `Webhooks` Zapier app for this step.
- Name this step "Get Token from Givetel" or similar.
- Set your url to Givetel's `/token` url from the documentation (currently `https://givetel-api.com/v2/token`)
- Set your `Payload Type` to `json`.
- In the `Headers` section, add an `x-api-key` header and paste your API key as the value for your header.
- Test your endpoint, you should receive a long token in the response along with an `Expires At` value.

### 8. Save the Token to Storage

- Select the `Storage` Zapier app for this step.
- Name the step "Store Token & Expiry" or similar.
- For the `Event` select `Set Multiple Values`.
- In the `Configure` tab, create a key called `token` and insert the `Data` from the output of the previous step. Additionally, create a key called `token_expires` and insert the `Expires At` from the output of the previous step.
- Test the step and ensure everything is stored successfully.

### 9: Return the token

- Select the `Sub-Zap` Zapier app for this step.
- Name the step "Return Token" or similar".
- For the `Event`, select `Return From a Sub-Zap`.
- In the `Configure` tab, create a return value called `token` and make the value the `Data` from the `Get New Token` step.

## 10: Fallback Path (No Token Refresh)

- In the right-hand branch after the `Split Into Paths` step, click the `Path Conditions` and set the condition to `Fallback`. We'll use this step when we have a token in the Zap's storage that hasn't expired.

### 11: Return the token that was fetched from storage

- Select the `Sub-Zap` Zapier app for this step.
- Name the step "Return Token" or similar.
- For the `Event`, select `Return From a Sub-Zap`.
- In the `Configure` tab, create a return value called `token` and make the value the `Data` from the `Get New Token` step.

# Tutorial Video

If you prefer guides of this nature in video format, we provide a walkthrough of the `/token`
setup at this url - https://www.youtube.com/watch?v=FUKk4nqBsrw
