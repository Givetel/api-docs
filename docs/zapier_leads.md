# /leads Zap Configuration

This is the Zap that will actually handle receiving the lead from a third party, processing that lead and sending it to the Givetel API. Compared to the `/leads` configuration, there will be a little more in this stage that you'll need to configure yourself, as we don't know the shape or source of the data that you'll be handling. However, this guide will give you enough of an idea to be able to set things up.

If you need to set up Zaps for data coming from multiple sources, with differing shapes, it will be easiest for you to set up a separate Zap for each discrete data source.

## 1. Receive the Data

- Select the `Webhooks` Zapier app for this step.
- Name this step "Receive Data" or similar.
- In the `Configure` tab, copy the `Webhook URL` and paste it into the service that will be sending you the leads.
- In the `Test` tab in Zapier, you should be able to listen for a test lead, which you want to trigger from your third party lead provider. Most providers will have a button where you can send test data.
- When you've sent a test record, you should be able to select it in your `Test` tab for this step. We'll use this record to set up the remaining steps of the Zap.

## 2. Validate/Format the Phone Number

- Select the `Formatter` Zapier app for this step.
- Name this step "Validate and Format Phone Number" or similar.
- For the `Event`, choose `Numbers`.
- In the `Configure` tab, open the `Transform` dropdown and select `Format Phone Number`.
- For your `Input`, select the main phone number field provided by your third party.
- For your `To Format` field, select the format with only numbers (no + signs, spaces or special characters) in International format.
- For `Phone Number Country Code`, select `AU`.
- For `Validate Phone Number`, select `Yes` - this will essentially reject leads with invalid phone numbers, which we want.
- Test your step and ensure it's working as expected.

## 3. Ensure there is an external ID

- Select the `Filter` Zapier app for this step.
- Name the step "Ensure External ID Exists" or similar.
- For the `Event` dropdown, select `Only continue if...`.
- In the `Configure` tab, use the value from your third party that uniquely identifies the record, and choose `exists` as your filter option.

## 4. Ensure there is a first name

- Select the `Filter` Zapier app for this step.
- Name the step "Ensure First Name Exists" or similar.
- For the `Event` dropdown, select `Only continue if...`.
- In the `Configure` tab, use the value from your third party that contains the supporter's first name, and choose `exists` as your filter option.

## 5. Ensure there is a last name

- Select the `Filter` Zapier app for this step.
- Name the step "Ensure First Name Exists" or similar.
- For the `Event` dropdown, select `Only continue if...`.
- In the `Configure` tab, use the value from your third party that contains the supporter's last name, and choose `exists` as your filter option.

## 6. Get your Authentication Token

- Select the `Sub-Zap` Zapier app for this step.
- Name the step "Get /token" or similar.
- Choose the `Call a Sub-Zap` Event.
- In the `Configure` tab, select your account and select the `/token` Sub-Zap that we set up previously.
- Test the step, it should return a token.

## 7. Create additional_data Object

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

## 8. Send the Lead to Givetel

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

# Video Walkthrough

If you prefer guides like this in video format, we provide a video walkthrough for the `/leads` Zap at this Youtube URL - https://www.youtube.com/watch?v=LAx0zqzTS08
