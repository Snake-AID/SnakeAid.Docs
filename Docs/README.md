# SnakeAid Documentation 🐍

> The official documentation for the AI-Powered Snakebite First Aid and Rescue Support System.

## Welcome!

Welcome to the **SnakeAid** documentation portal. This site provides comprehensive guides, specifications, and architecture references for the entire SnakeAid ecosystem.

Our mission is to save lives by providing timely first aid guidance, AI-based snake identification, and efficient rescue coordination.

## Preview locally

Run Docsify from the repository root:

```bash
npx docsify-cli@latest serve Docs
```

Then open the served URL (defaults to `http://localhost:3000`) in your browser.

If you prefer a reusable install, run `npm install -g docsify-cli` once, then use `docsify serve Docs`.


## 📚 Documentation Sections

Explore our documentation by category:

| Section | Description | Audience |
|---------|-------------|----------|
| **[00. General](00-General/Project-Overview.md)** | Project overview, scope, and high-level goals. | Everyone |
| **[01. Functional Specs](01-Functional-Specs/00-Concepts/Feature-Matrix.md)** | Detailed requirements for all user roles (Patient, Rescuer, Expert, Admin). | PMs, Developers |
| **[02. Architecture](02-Technical-Architecture/00-System-Context.md)** | System context, swimlane diagrams, and technical design. | Architects, Lead Devs |
| **[03. Data Design](03-Data-Design/00-SnakeAid/SnakeAid.md)** | Entity Relationship Diagrams (ERDs) and schema details. | Backend Devs, DBAs |
| **[04. Frontend](04-UI-UX/00-Overview.md)** | UI/UX designs, wireframes, and mobile app flows. | Frontend Devs, Designers |
| **[05. Backend](05-Backend/HOME.md)** | API documentation, service layer details, and backend guides. | Backend Devs |
| **[06. Review](06-Review/R01/Note.md)** | Meeting notes, mentor feedback, and review reports. | Team |

## 🚀 Key Features of this Docs Site

*   🔍 **Search**: Use the search bar in the top left to find any topic.
*   🌗 **Dark Mode**: Switch between light and dark themes for comfortable reading.
*   📱 **Responsive**: Access documentation easily on mobile or desktop.
*   📊 **Diagrams**: Rich visualizations using Mermaid and PlantUML.
*   📋 **Copy Code**: Easily copy code snippets and examples.

## 🛠️ Contribution Guidelines

We welcome contributions! To update documentation:

1.  **Edit Markdown**: Files are located in the `Docs/` directory.
2.  **Use Standards**: Follow the [Backend Documentation Guide](05-Backend/README.md) for structure.
3.  **Update Sidebar**: If adding new pages, update `_sidebar.md`.
4.  **Preview**: Run `docsify serve Docs` locally to verify changes.

---

**Project**: SnakeAid  
**Last Updated**: 2026-02-03  
**Maintained By**: SnakeAid Team
