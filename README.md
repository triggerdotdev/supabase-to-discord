## ✨ Trigger.dev Supabase to Discord

This repo uses [Supabase Webhooks](https://supabase.com/docs/guides/database/webhooks) with the generic [webhookEvent](https://docs.trigger.dev/reference/webhook-event) trigger to send updates whenever a record is created in a Supabase table to Discord, using [Discord incoming webhooks](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) and our generic [fetch](https://docs.trigger.dev/functions/fetch) function.

```ts
import { Trigger, webhookEvent } from "@trigger.dev/sdk";

const SUPABASE_TABLE = process.env.SUPABASE_TABLE ?? "users";
const DISCORD_WEBHOOK_URL = process.env.DISCORD_WEBHOOK_URL;

new Trigger({
  // Give your Trigger a stable ID
  id: "supabase-to-discord",
  name: "Supabase to Discord",
  // Trigger on a webhook event, see https://docs.trigger.dev/triggers/webhooks
  on: webhookEvent({
    service: "supabase", // this is arbitrary, you can set it to whatever you want
    eventName: "row.inserted", // this is arbitrary, you can set it to whatever you want
    filter: {
      type: ["INSERT"], // only trigger on INSERT events
      table: [SUPABASE_TABLE], // only trigger
    },
  }),
  // The payload of the webhook is passed as the event argument
  async run(event, ctx) {
    // Check that the DISCORD_WEBHOOK_URL environment variable is set
    if (!DISCORD_WEBHOOK_URL) {
      throw new Error("DISCORD_WEBHOOK_URL is not set");
    }

    // Craft a message to send to Discord
    const message = {
      embeds: [
        {
          color: 0x7289da,
          title: `New row created in ${event.table}: ${event.record.email}`,
        },
      ],
    };

    // Send the message to Discord using the incoming webhook URL
    await ctx.fetch("✨", DISCORD_WEBHOOK_URL, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: message,
    });
  },
}).listen();
```

## 🔧 Install

You can easily create a new project interactively based on this template by running:

```sh
npx create-trigger@latest supabase-to-discord
# or
yarn create trigger supabase-to-discord
# or
pnpm create trigger@latest supabase-to-discord
```

Follow the instructions in the CLI to get up and running locally in <30s.

## 💿 Generate Discord Webhook URL

Follow the instructions in [this Discord article](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) for how to generate a Discord Webhook URL for your Discord Server.

Once you have the URL from Discord, set the `DISCORD_WEBHOOK_URL` env variable in the `.env` file at the root of this project:

```
DISCORD_WEBHOOK_URL="your discord webhook url here"
```

## ⚡️ Register Supabase Webhook

Once you complete the CLI command, you'll need to register the webhook with Supabase to get events flowing. Start by visiting your [Trigger.dev dashboard](https://app.trigger.dev) and navigating to the "Supabase to Discord" workflow you just created:

![Workflow List](https://imagedelivery.net/3TbraffuDZ4aEf8KWOmI_w/4129c7ea-f5fb-4644-8d11-90462c2e2500/public)

Click on the list item and you should see the webhook URL you'll need. Copy it:

![Workflow Overview Start](https://imagedelivery.net/3TbraffuDZ4aEf8KWOmI_w/7c84b018-fabc-479e-e1c6-e1f3e79d1400/public)

Open up your Supabase Project and navigate to "Database" -> "Webhooks ALPHA":

![Webhooks Alpha](https://imagedelivery.net/3TbraffuDZ4aEf8KWOmI_w/43434c3b-0781-4ece-d16d-9b8068ccb700/public)

Create a new webhook and fill out the form, entering the URL you just copied from Trigger.dev, choosing the "Insert" event, select "HTTP Request", and make sure it's a POST (not a GET):

![Webhook Form](https://imagedelivery.net/3TbraffuDZ4aEf8KWOmI_w/2a1a9e33-fd03-45c4-cbcf-eef332e26600/public)

After creating the webhook you can now insert a new record into your database and when you refresh the Workflow Overview page in Trigger.dev you should see a new completed run:

![New Run](https://imagedelivery.net/3TbraffuDZ4aEf8KWOmI_w/aa427b97-dc5d-4c44-52b3-a9e03b64ab00/public)

Click on the run and you can see the event that was sent to your `Trigger.run` function and view the request to the Discord Webhook URL:

![Run](https://imagedelivery.net/3TbraffuDZ4aEf8KWOmI_w/a38ad1f4-0c7f-4c80-f060-cf49c7bfe000/public)

And checking Discord, we should see our newly sent message:

![Discord](https://imagedelivery.net/3TbraffuDZ4aEf8KWOmI_w/a489ce26-d3f1-40b7-be15-ee56d32b6500/public)

## ✍️ Customize

1. Change the table using the `SUPABASE_TABLE` environment variable to point to the table you'd like events from in your supabase database.
2. Feel free to customize the message to Discord by referencing [their docs](https://discord.com/developers/docs/resources/webhook#execute-webhook)
