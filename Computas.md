In the summer of 2025 I interned at [Computas](https://computas.com/en/), building an AI assistant for [NRK TV-Aksjonen](https://tvaksjonen.no/), Norway's largest annual fundraising campaign. Computas is a Norwegian IT consultancy, and the internship was one of 16 spots with more than 800 applicants; [Kode24 wrote about the group](https://www.kode24.no/artikkel/822-sokte-her-er-de-16-som-fikk-sommerjobb-hos-computas/239270). We were a team of three, all computer science students from NTNU.

## The problem

TV-Aksjonen runs on temporary staff through its campaign season, and every autumn a new wave of them meets the same wall: the answers to their practical questions all exist, sick leave, salary procedures, daily routines among them, but they sit in SharePoint, written for people who already know the organization. The internal abbreviations make it worse, since a document is no help when its title is three letters you have never seen. So the questions went to the permanent staff instead, one interruption at a time.

## A chatbot where the questions already were

The staff lived in Microsoft Teams and reached SharePoint through it, so the assistant became a Teams app rather than one more website to remember. My side of the project was the backend and the Azure infrastructure; my two teammates built the frontend, a React and TypeScript app wired into Teams through the Microsoft 365 Agents Toolkit, with sign-in through Entra ID.

The retrieval pipeline connects TV-Aksjonen's SharePoint to Azure AI Search through Microsoft Graph connectors. I configured the indexers and skillsets that break the PDF and Word documents into chunks, and the search is hybrid: semantic embeddings and classic keyword matching side by side, so a question finds the right document whether or not it happens to use the document's own words. When a question comes in, an Azure Function in Python validates the user's Teams login, gathers the relevant chunks together with the FAQs, the abbreviation definitions and the conversation so far, and sends it all to Azure OpenAI. The answer comes back with its sources attached, so a reader can check the claim instead of having to trust the bot.

All of it stands as code: Terraform for the infrastructure, GitHub Actions for deployment and state, so the system can be rebuilt, or handed to someone else, without archaeology.

## Handing over the controls

The part I think mattered most was the administrative interface. TV-Aksjonen wanted control over the system's behaviour, so non-technical staff can edit the FAQs, maintain the abbreviation list, and adjust the prompts that steer the model. Logging shows which questions the assistant struggles with, so the people who own the knowledge can see the gaps and fill them. The goal was a system its owners could run and improve after we were gone.

At the end of the summer we handed the system over. TV-Aksjonen was very happy with it and said they would use it through the autumn campaign, and new hires at Computas carried the last stretch into production after we left. How it fared through the campaign I do not actually know: consulting means leaving before the verdict. I think, and hope, it is still answering questions.

## What it taught me

I came in from [half a year of Google Cloud at Q-Free](https://pages.askhb.no/Q-Free) and left with the Azure counterpart of that map: AI Search for retrieval, role-based access control where I would earlier have reached for API keys, and Entra ID with multi-tenant application registration, the part of the stack that took the longest to actually understand. The consulting side was its own lesson. Six weeks from first commit to handover leaves no room for architecture you cannot defend, and the features that survived were the ones a temporary employee with a question would actually touch.
