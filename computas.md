In the summer of 2025, I completed an internship at [Computas](https://computas.com/en/), a leading Nordic IT consultancy. I was selected as one of 16 students from over 800 applicants, working alongside fellow students from various universities and disciplines. More details about our group can be found i [this Kode24 article](https://www.kode24.no/artikkel/822-sokte-her-er-de-16-som-fikk-sommerjobb-hos-computas/239270). 

## The Project: AI-Powered Knowledge Assistant for NRK TV-Aksjonen
I worked on developing a context-aware AI chatbot for [NRK TV-Aksjonen](https://tvaksjonen.no/), Norway's largest annual fundraising campaign. Our three-person team, all Computer Science students from NTNU, tackled what I considered the most challenging and rewarding project among the four intern initiatives.
## The Problem We Solved
TV-Aksjonen relies heavily on temporary staff during their autumn and winter campaign season. These employees frequently struggled to locate essential information in SharePoint about sick leave policies, salary procedures, operational guidelines, and other internal processes. This resulted in constant interruptions to permanent staff members, creating significant productivity bottlenecks.

The challenge was compounded by TV-Aksjonen's extensive use of internal abbreviations, which created an additional barrier for new employees trying to navigate documentation and procedures. What we needed was a solution that could bridge this knowledge gap while being intuitive and accessible to temporary workers.

## Our Solution
We decided early on that the solution needed to live where employees already spent their time. Since TV-Aksjonen used Microsoft Teams for internal communication and typically accessed SharePoint through Teams, building a native Teams application was the natural choice. This approach minimized friction and ensured high adoption rates.

I focused primarily on the backend infrastructure and Azure setup. Using Terraform for infrastructure as code, I deployed all our Azure resources with GitHub Actions handling CI/CD and state management. This gave us a robust, repeatable deployment process that TV-Aksjonen could maintain long-term.

The core of our system was built around connecting TV-Aksjonen's SharePoint to Azure AI Search through Microsoft Graph connectors. I configured custom indexers and skillsets to process PDF and DOCX documents, breaking them into smaller chunks for better search accuracy and performance. We implemented hybrid search capabilities, combining semantic vector embeddings with traditional keyword indexing to ensure users could find relevant information regardless of how they phrased their questions.

My teammates handled the frontend development, building a React TypeScript application with Vite. They integrated the Microsoft 365 Agents Toolkit to ensure seamless Teams integration and proper authentication through Entra ID. The frontend was hosted on Azure Static Web Apps, serving the static pages that the Teams application requested.

For the backend services, I built serverless APIs using Azure Functions in Python. The authentication flow used JWT tokens from Entra ID, validating users through their existing Teams login. When users asked questions, the system would search Azure AI Search for relevant SharePoint content, combine it with FAQs and abbreviation definitions, include conversation history for context, and send everything to an OpenAI endpoint. The response included both the AI-generated answer and source attribution so users could verify the information.

One feature I'm particularly proud of is the administrative interface we built. TV-Aksjonen wanted maximum control over the system's behavior, so we created tools that allowed non-technical staff to manage FAQs, update abbreviation definitions, and even customize the AI's guiding prompts. This meant the system could improve over time as administrators identified knowledge gaps and added new content.

We also implemented logging so administrators could see what questions the chatbot struggled with, enabling them to continuously refine the knowledge base and improve the user experience.

## Technologies and Architecture
The entire solution ran on Azure infrastructure, leveraging Microsoft Graph connectors for SharePoint integration, Azure AI Search for document retrieval, Azure OpenAI for natural language processing, and Azure Functions for serverless compute. The frontend used React with TypeScript, built with Vite and integrated through the Microsoft 365 Agents Toolkit. We stored configuration data, FAQs, abbreviations and chat conversations in Azure Blob Storage, and everything was deployed using Terraform and GitHub Actions.

## What I Learned
This internship deepened my understanding of enterprise-scale AI system development and introduced me to the Azure ecosystem. While I had previous experience building AI systems on Google Cloud, working with Azure's suite of services gave me valuable perspective on different cloud architectures and approaches. I gained hands-on experience with Azure's Role-Based Access Control as a more secure alternative to API keys, learned to navigate the complexities of Entra ID and multi-tenant application registration, and discovered how Terraform can streamline infrastructure management across different cloud environments.

Perhaps most importantly, I experienced what it's like to work in a fast-paced consulting environment where you need to deliver real business value quickly. Building a production-ready system in just six weeks taught me the importance of making smart architectural decisions early and focusing on features that directly address user needs.



 