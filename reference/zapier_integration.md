---
title: Givetel Zapier Integration
---

# Zapier Integration

In your Zaps, create a folder called `Givetel` and in that folder, create two Zaps: `/token` and `/leads`. We need to configure both to create leads in the Givetel system. Make sure you configure the `/token` Zap _first_ for the smoothest experience.

## /token Zap Configuration

This is a Sub-Zap, which is a Zap that can be called by another Zap. This is useful when you need a utility that can be used across many Zaps if needed, which is the case for authentication endpoints like `/token`.

If you prefer guides of this nature in video format, we provide a walkthrough of the `/token`
setup at this url - https://www.youtube.com/watch?v=FUKk4nqBsrw

The following are the steps involved in setting up the `/token` Sub-Zap. Each heading will represent a new step in the Zap.

### 1. Create a Trigger - Sub-Zap by Zapier

- Simply select the Sub-Zap by Zapier app for this step and click continue through until you can run a test and get a test record. This record will be blank and we won't do anything with it.

### 2. Get Token & Expiry If Exists

- Select the Storage by Zapier app for this step. You may be prompted to set this app up for your account in this step.
- Name this step "Get Token & Expiry From Storage" or similar.
- Choose the Action Event `Get Multiple Value` and press Next.
- In the `Configure` tab, enter the keys `token` and `token_expires`.

### 3. Get The Current Time in UTC

- Select the `Code` Zapier app for this step.
- Name this step "Get Current Timestamp" or similar.
- Select the action event `Run Javascript`.
- Use the following code in the `Configure` tab:

```javascript
return { now: new Date().toISOString() };
```

- Test the step and test that the current time in UTC is being returned.

### 4. Check if Local Token is Expired

- Select the `Formatter` Zapier app for this step.
- Name this step "Has Token Expired?" or similar.
- Select the action event `Date/Time`.
- In the `Configure` tab, set the `Start Date` to the current date & time value from the previous step and set the `End Date` to the value from step 2. For the `Date Format - Start Date` you want to use `YYYY-MM-DDTHH:mm:ssZ` (note the T between the DD and the HH), and for the `Date Format - End Date` you want to use `YYYY-MM-DD HH:mm:ss Z` (note the space beteen the DD and the HH).

### 5. Split Into Paths

- Select the `Paths` Zapier app for this step.

### 6. Set Path Conditions for Refresh Token

- In the left-hand side of the path you've created, click `Path Conditions` and set the type of your rules to `Custom Rules`.
- In the `Only continue if` section, select the `Token` value from step 2 and select `Does not exist`.
- Click `Add "Or" rule group`, and configure the new condition to cntinue if the `Output Dates Swapped` value from step 4 is true.

#### 7. Get New Token

- Select the `Webhooks` Zapier app for this step.
- Name this step "Get Token from Givetel" or similar.
- Set your url to Givetel's `/token` url from the documentation (currently `https://givetel-api.com/v2/token`)
- Set your `Payload Type` to `json`.
- In the `Headers` section, add an `x-api-key` header and paste your API key as the value for your header.
- Test your endpoint, you should receive a long token in the response along with an `Expires At` value.

#### 8. Save the Token to Storage

- Select the `Storage` Zapier app for this step.
- Name the step "Store Token & Expiry" or similar.
- For the `Event` select `Set Multiple Values`.
- In the `Configure` tab, create a key called `token` and insert the `Data` from the output of the previous step. Additionally, create a key called `token_expires` and insert the `Expires At` from the output of the previous step.
- Test the step and ensure everything is stored successfully.

#### 9: Return the token

- Select the `Sub-Zap` Zapier app for this step.
- Name the step "Return Token" or similar".
- For the `Event`, select `Return From a Sub-Zap`.
- In the `Configure` tab, create a return value called `token` and make the value the `Data` from the `Get New Token` step.

### 10: Fallback Path (No Token Refresh)

- In the right-hand branch after the `Split Into Paths` step, click the `Path Conditions` and set the condition to `Fallback`. We'll use this step when we have a token in the Zap's storage that hasn't expired.

#### 11: Return the token that was fetched from storage

- Select the `Sub-Zap` Zapier app for this step.
- Name the step "Return Token" or similar".
- For the `Event`, select `Return From a Sub-Zap`.
- In the `Configure` tab, create a return value called `token` and make the value the `Data` from the `Get New Token` step.

## /leads Zap Configuration

This is the Zap that will actually handle receiving the lead from a third party, processing that lead and sending it to the Givetel API. Compared to the `/leads` configuration, there will be a little more in this stage that you'll need to configure yourself, as we don't know the shape or source of the data that you'll be handling. However, this guide will give you enough of an idea to be able to set things up.

If you prefer guides like this in video format, we provide a video walkthrough for the `/leads` Zap at this Youtube URL - https://www.youtube.com/watch?v=LAx0zqzTS08

If you need to set up Zaps for data coming from multiple sources, with differing shapes, it will be easiest for you to set up a separate Zap for each discrete data source.

### 1. Receive the Data

- Select the `Webhooks` Zapier app for this step.
- Name this step "Receive Data" or similar.
- In the `Configure` tab, copy the `Webhook URL` and paste it into the service that will be sending you the leads.
- In the `Test` tab in Zapier, you should be able to listen for a test lead, which you want to trigger from your third party lead provider. Most providers will have a button where you can send test data.
- When you've sent a test record, you should be able to select it in your `Test` tab for this step. We'll use this record to set up the remaining steps of the Zap.

### 2. Validate/Format the Phone Number

- Select the `Formatter` Zapier app for this step.
- Name this step "Validate and Format Phone Number" or similar.
- For the `Event`, choose `Numbers`.
- In the `Configure` tab, open the `Transform` dropdown and select `Format Phone Number`.
- For your `Input`, select the main phone number field provided by your third party.
- For your `To Format` field, select the format with only numbers (no + signs, spaces or special characters) in International format.
- For `Phone Number Country Code`, select `AU`.
- For `Validate Phone Number`, select `Yes` - this will essentially reject leads with invalid phone numbers, which we want.
- Test your step and ensure it's working as expected.

### 3. Ensure there is an external ID

- Select the `Filter` Zapier app for this step.
- Name the step "Ensure External ID Exists" or similar.
- For the `Event` dropdown, select `Only continue if...`.
- In the `Configure` tab, use the value from your third party that uniquely identifies the record, and choose `exists` as your filter option.

### 4. Ensure there is a first name

- Select the `Filter` Zapier app for this step.
- Name the step "Ensure First Name Exists" or similar.
- For the `Event` dropdown, select `Only continue if...`.
- In the `Configure` tab, use the value from your third party that contains the supporter's first name, and choose `exists` as your filter option.

### 5. Ensure there is a last name

- Select the `Filter` Zapier app for this step.
- Name the step "Ensure First Name Exists" or similar.
- For the `Event` dropdown, select `Only continue if...`.
- In the `Configure` tab, use the value from your third party that contains the supporter's last name, and choose `exists` as your filter option.

### 6. Get your Authentication Token

- Select the `Sub-Zap` Zapier app for this step.
- Name the step "Get /token" or similar.
- Choose the `Call a Sub-Zap` Event.
- In the `Configure` tab, select your account and select the `/token` Sub-Zap that we set up previously.
- Test the step, it should return a token.

### 7. Create additional_data Object

- Select the `Code` Zapier app for this step.
- Name the step "Create additional_data" or similar.
- Select the `Run Javascript` event.
- In the `Configure` tab, paste in this Javascript block. Don't worry about filling out any data for now.

```javascript
// This should only ever include values that can't fit into Givetel's native API fields
// The native API fields are always preferred, because Givetel's automated reporting system is designed to work with them by default
// However, there's no way to guarantee Givetel's native fields will cover every possible piece of data coming out of 3rd party services. As such, we provide `additional_data` for these outlier cases.
return {
  additional_data: JSON.stringify({}),
};
```

- Test your step, it should run successfully.

### 8. Send the Lead to Givetel

- Select the `Webhooks` app for this step.
- Name the step "POST lead to Givetel" or similar.
- Choose the `POST` Event.
- In the `Configure` tab, set our `/leads` endpoint as the URL (currently the value is `https://givetel-api.com/v2/leads`).
- Select the `json` Payload Type.
- At the bottom of the page, in the `Headers` section, create an `x-api-key` header like in the `/token` Zap and paste in your api key as the value.
- Create an additional header with the key `Authorization`. For the value, you should type `Bearer ` (note the space after Bearer), and then you should insert the `token` value from the `Get /token` step. The final value for this header should be something like `Bearer eyJhbGc......` with the remainder of the long token existing here.
- Using the Givetel documentation as a guide, map the fields in the lead you're processing from the record's default fields to Givetel's native fields in the `Data` section of the configuration tab of this record. As much as possible, it is ideal to map fields to Givetel's native fields so they can be better utilized on our end.
- When entering the `phone` field in your data, make sure to take the output of the `Validate & Format Phone Number` step so you provide a correctly formatted phone number.
- If your test record has fields that can't be easily mapped to Givetel's native fields, add those into your `additional_data` object in the previous step.
- If you have added fields to `additional_data` make sure to include the `additional_data` key in the Data for this step's configuration and make the value of `additional_data` the output of the previous step.
- Test your Zap and ensure you're getting a response from Givetel. You may get a `403 - Invalid Authentication Token` to start with. This is okay, and is a result of the way Sub-Zaps work during testing. If you get any other error, you may have configured something incorrectly and should follow the direction of the error you receive.
- If your test has succeeded, or failed with a `403 - Invalid Authentication Token`, publish your Zap and trigger a new test lead. You should be able to go into your `Zap History` from the Zapier homepage to monitor the performance. If the Zap succeeds, you're set up and ready to go!
