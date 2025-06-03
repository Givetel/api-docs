---
title: Givetel Zapier Integration
---

# Zapier Integration

In your Zaps, create a folder called `Givetel` and in that folder, create two Zaps: `/token` and `/leads`. We need to configure both to create leads in the Givetel system. Make sure you configure the `/token` Zap _first_ for the smoothest experience.

The `token` endpoint is used to create a token that can be used to authenticate subsequent requests to the `leads` endpoint. See the [/token Zap Configuration](zapier_token.md) for more details.

The `leads` endpoint is used to create a new lead in the Givetel system. See the [/leads Zap Configuration](zapier_leads.md) for more details.
