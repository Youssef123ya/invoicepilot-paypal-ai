# InvoicePilot

Turn freelancer work notes into a reviewed invoice, then create and track it in **PayPal Sandbox**. Built for the PayPal AI Hackathon.

## Current status

The application and provider integrations are implemented. **Live tests passed for Gemini structured drafting, missing-detail clarification, hosted human review, PayPal sandbox invoice creation, payment-link activation, and status refresh.** The generated example totals $600.00 USD. The activated invoice remains UNPAID: no buyer payment has been completed or verified. Automated provider tests use mocks. The prepared sample is clearly labeled and never pretends to be a provider result. This project has not been sent to Devpost.

## Source package

Download and extract **InvoicePilot-source.zip** from this repository. It contains the complete application source, assets, MIT license, dependency lockfile, and tests. Run the commands below inside the extracted folder. The archive excludes credentials and generated dependencies.

Demo video: https://youtube.com/shorts/KYOtxpa2Lqk

## Run locally on Windows

Requires Node.js 24+ and npm. From this project directory:

```powershell
npm ci
powershell -ExecutionPolicy Bypass -File scripts/setup-connections.ps1
npm run dev
```

Open the printed Local URL (normally `http://127.0.0.1:5173`). For connected actions, use the portable starter's local sign-in at `http://127.0.0.1:5173/signin-with-chatgpt?return_to=/`. This local simulated sign-in is development-only; hosted access uses Sites authentication.

Without credentials you can still try the prepared sample, edit line items, save/restore a browser-local draft, and export invoice JSON. AI drafting and PayPal operations fail clearly until configured.

### Account setup

1. Sign in to [PayPal Developer](https://developer.paypal.com/dashboard/). In **Apps & Credentials**, select **Sandbox** and create an app. Keep its Client ID and Secret private. In Sandbox Accounts, identify a business seller and personal buyer.
2. Sign in to [Google AI Studio](https://aistudio.google.com/apikey), create an API key, and review model access, quotas, and billing settings. The model defaults to `gemini-3.5-flash-lite`; change `GEMINI_MODEL` to an accessible model supporting structured output if needed.
3. Run `scripts/setup-connections.ps1`. It asks for all three values using hidden prompts and writes ignored `.dev.vars` for local Worker development. Restart the server after changes. Never commit this file or paste credentials into chat.
4. Hosted secrets must be configured separately in Sites runtime settings: `PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`, `GEMINI_API_KEY`, optionally `GEMINI_MODEL`. Redeploy to apply a new environment revision. Local `.dev.vars` is never uploaded. Keep the hosted site private while it uses a shared seller account.

## Workflow

1. Enter work notes with client, email, line items, explicit quantities and rates.
2. Choose a currency and **Draft with AI**. Gemini returns structured JSON and questions about missing facts. It cannot create invoices or send payments. Review every extracted amount and detail; model output is not guaranteed to be correct.
3. Edit the business name, client, quantities, prices, terms, and note. Missing prices/quantities remain incomplete rather than being invented. Only USD, EUR and GBP are supported in this version; all use two decimal places. Taxes, discounts, attachments, tips, and partial payments are out of scope.
4. Check the review box and create the PayPal sandbox draft. The payload is validated again server-side. A stable invoice number and request header are reused for retries; after an uncertain network outcome, refresh the invoice list/check the PayPal dashboard before starting another invoice.
5. Activate the sandbox payment link after reviewing its confirmation dialog. PayPal email notifications are disabled. Open the link using the **personal sandbox buyer**, complete the test payment, and refresh the invoice status. The app never marks an invoice paid on its own.

## Technology and architecture

- React, TypeScript, Vinext/Vite, Cloudflare Workers, Zod, Lucide icons.
- Gemini generateContent REST API: work notes → structured draft JSON using stateless requests; API key stays server-side. Stateless requests do not replace the provider's published data handling terms.
- PayPal Invoicing v2: OAuth client credentials, create draft, list invoices, activate link, read invoice status. The base URL is fixed to `https://api-m.sandbox.paypal.com`; live PayPal is not supported.
- Sites private hosting and authentication. Provider routes require a signed-in user. Writes check same-origin requests; error responses redact provider payloads.
- PayPal is the source of truth for invoices; no application database is used. **Invoices** lists the seller's 30 most recent sandbox invoices. Work-in-progress persistence is opt-in localStorage on the current browser/device, and JSON export.

Entry points: `app/invoice-app.tsx` (UI), `app/api/[...path]/route.ts` (API and authorization), `lib/invoice.ts` (validation/money/PayPal mapping), `lib/services.ts` (providers). Preserve the starter's Sites build integration when deploying. Runtime secrets are not in `.openai/hosting.json`.

## Verification

```powershell
node --experimental-transform-types --test tests/invoice.test.ts
node node_modules/typescript/bin/tsc --noEmit
npm run build
```

Automated tests cover line-level currency arithmetic, input validation, sandbox URL restrictions, missing credential behavior, OAuth and request mapping, link-only create responses, redacted errors, and structured AI output. Provider tests use mocks and do not prove live account compatibility.

Manual acceptance checklist after connecting accounts:

- [x] Live AI extracts the example work notes into two rows and a $600.00 USD total.
- [x] Missing quantities/rates produce null fields and clarification questions.
- [x] Editing resets the hosted human review control and disables creation.
- [x] Real sandbox invoice creation and API readback match the reviewed amounts.
- [x] The activated link opens PayPal's sandbox invoice with payment controls; notifications are disabled.
- [x] Refresh reads the actual UNPAID state from PayPal.
- [ ] Complete a sandbox buyer payment and verify PAID readback.
- [ ] Repeated create attempts do not create unintended duplicate invoices.
- [ ] Mobile layout, local save/restore, JSON export, errors, and authentication work.

## Hackathon preparation still required

Publish this source to a **public** GitHub/GitLab/Bitbucket repository with this MIT license and setup instructions, and provide a judge-runnable build or hosted demo. The current Sites deployment is owner-private and is not judge access. The Sites source repository/private preview is not a substitute for the required public repository. Do not publish API secrets; judges can configure their own sandbox and Gemini credentials using the setup instructions.

An English demo was scripted, narrated in ElevenLabs, and edited/exported on this device in Wondershare Filmora. The MP4 is 1 minute 59 seconds, 1080 × 1080, 25 fps, with AAC narration. It demonstrates the working app and makes no paid-invoice claim. Public YouTube upload and final Devpost confirmation remain.

## Official integration references

- [PayPal Invoicing v2](https://developer.paypal.com/api/invoicing/v2)
- [PayPal Sandbox accounts](https://developer.paypal.com/sandbox-testing/accounts)
- [Gemini structured output](https://ai.google.dev/gemini-api/docs/structured-output)
- [Gemini getting started](https://ai.google.dev/gemini-api/docs/get-started)

## License

Application source is MIT licensed. Third-party packages and the vendored Sites integration retain their own licenses.




