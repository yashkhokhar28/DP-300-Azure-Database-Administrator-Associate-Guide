## 🌟 **Key Idea: Two Deployment Models**

### 1. **Declarative Model (Preferred for Large Deployments)**

- **Example tools:** Azure Resource Manager (**ARM**) templates, **Bicep**, Azure portal export.
- **Bicep is Microsoft's recommended authoring language**; Bicep files compile to ARM JSON at deployment time. 🔄
- **How it works:**

  - You **describe the final state** of your infrastructure (**what you want**), not the step-by-step process.
  - Azure **figures out the order** to create and configure your resources.
  - Supports **dependencies and orchestration** (automatically creates dependent resources in the correct order).
  - Example: Deploy a Virtual Machine, its Network, and Storage in one go.

  ✅ **Advantages:**

  - Consistent and repeatable deployments.
  - Easier to version-control (stored in source control like GitHub).
  - Scales well for complex environments.

---

### 2. **Imperative Model (Procedural Approach)**

- **Example tools:** PowerShell, Azure CLI.
- **How it works:**

  - You **write scripts** that specify **each step** to create resources.
  - Execution happens **step by step**, in the order you wrote.

  ✅ **Advantages:**

  - Easier for **small tasks or quick changes**.
  - Can be used for **modifying or checking** existing resources.

---

## 🔧 **Deployment Tools Explained**

### ▶ **Azure Resource Manager (ARM) Templates**

- JSON files that **declare your infrastructure**.
- Support **parameters**, so you can reuse the same template with different values (e.g., region, resource size).
- Can run **custom scripts** (PowerShell, Bash) after resource creation.
- Deployed via:

  - Azure Portal (graphical)
  - Azure CLI
  - PowerShell
  - Azure DevOps Pipelines

---

### ▶ **PowerShell**

- Uses the **Az module** (Az.Compute, Az.Sql, etc.).
- Suitable for:

  - Resource modifications
  - Status checks
  - Simple deployments
  - Deploying ARM templates

---

### ▶ **Azure CLI**

- Similar to PowerShell but works across OSes (Windows, Linux, macOS).
- Can perform:

  - Resource creation, modification, and deletion
  - Deploying ARM templates

- Some commands for **Azure Database for PostgreSQL and MySQL** are **CLI-exclusive**. 🔄

---

### ▶ **Azure Portal**

- **Graphical interface** for resource management.
- Allows **Exporting ARM templates** for automation later.
- Convert an exported ARM JSON template to Bicep with **`bicep decompile`** (`az bicep decompile`). 🔄
- Good for beginners and learning resource structure.

---

### ▶ **Azure DevOps Services (Azure Pipelines)**

- Automates deployments using **Continuous Integration (CI)** and **Continuous Delivery (CD):**

  - **CI:** Automates building, testing, and storing code changes.
  - **CD:** Automates deploying the latest changes to your Azure infrastructure.

- Uses ARM templates or scripts to deploy consistently and reliably.

---

## ✅ **Why It Matters**

- Declarative deployments (**ARM Templates, Bicep**) = **repeatable, error-free infrastructure setups**.
- Imperative deployments (**PowerShell, CLI**) = **flexible, scriptable operations**.
- Azure Pipelines + ARM Templates = **fully automated CI/CD pipelines**, reducing human error and improving efficiency.

---
