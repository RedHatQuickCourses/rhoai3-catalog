# Red Hat OpenShift AI (RHOAI) 3.2 Model Catalog - Operations Guide

**From Choice Overload to Confident Deployment**

> **The Problem:** Users can find models anywhere. They cannot safely deploy them everywhere.  
> **The Solution:** The Model Catalog provides a curated, validated “showroom” with a streamlined deployment wizard and enterprise-ready artifacts (ModelCars).

This repository contains a complete **Course-in-a-Box** that teaches platform engineers and administrators how to operate the **Model Catalog** in **Red Hat OpenShift AI 3.3**: architecture, deployment workflow, source configuration, and known constraints.

📚 **Full course documentation:** The complete Antora course (architecture, labs, troubleshooting) is in this repository—see **Option 1: View the Full Course** below to build and view it. Use the course when you need more detail than this runbook.

---

## 📚 Option 1: View the Full Course (Antora)

### Using Docker (Recommended)

```bash
docker run -u $(id -u) -v $PWD:/antora:Z --rm -t antora/antora antora-playbook.yml
# Open the generated site:
# open build/site/index.html
```

### Using Local NPM

```bash
npm install
npx antora antora-playbook.yml
# Open build/site/index.html
```

---

## ⚡ Option 2: The Fast Track (Admin Runbook)

If you already know the concepts and just need the operational commands, use this runbook.

### Prerequisites

* **Cluster:** OpenShift AI 3.3 installed and accessible
* **Access:** Permissions to view/edit resources in `rhoai-model-registries` (or `cluster-admin`)
* **CLI:** `oc` installed and authenticated (`oc login`)
* **Repository (optional):** Clone this repo to apply config as code: `git clone https://github.com/RedHatQuickCourses/rhoai3-catalog.git && cd rhoai3-catalog`

---

## Step 1: Confirm the Catalog is Visible in the Dashboard

The Model Catalog is enabled by default, but it can be hidden via `OdhDashboardConfig`.

```bash
oc get odhdashboardconfig -n redhat-ods-applications
```

To **enable** the catalog menu:

```bash
oc patch odhdashboardconfig odh-dashboard-config -n redhat-ods-applications --type merge -p '{
  "spec": { "dashboardConfig": { "disableModelCatalog": false } }
}'
```

---

## Step 2: Inspect and Manage Catalog Sources

The catalog reads sources from:

* **Namespace:** `rhoai-model-registries`
* **ConfigMap:** `model-catalog-sources`

```bash
oc get configmap model-catalog-sources -n rhoai-model-registries -o yaml
```

**Managing sources as code:** The ConfigMap YAML is not in this repo—you create or obtain it in one of these ways:

* **Export from the cluster** (if the ConfigMap already exists):  
  `oc get configmap model-catalog-sources -n rhoai-model-registries -o yaml > model-catalog-sources.yaml`  
  Edit the file, then apply: `oc apply -f model-catalog-sources.yaml`
* **Use a template for a new setup:** The Antora course in this repo and the [rhoai3-registry](https://github.com/RedHatQuickCourses/rhoai3-registry) repo (`deploy/catalog/catalog-source.yaml`) show the required ConfigMap structure and example entries. Create your own file (e.g. `model-catalog-sources.yaml`) from that, then:

```bash
oc apply -f model-catalog-sources.yaml
```

### Force a Reload (Recommended After Changes)

```bash
oc delete pod -l component=model-catalog -n rhoai-model-registries
oc get pods -l component=model-catalog -n rhoai-model-registries -w
```

---

## Step 3: Deploy from the Catalog (Wizard Guardrails)

When deploying from **AI hub → Catalog** in RHOAI 3.3:

* **Prefer ModelCars:** OCI artifacts hosted on `registry.redhat.io` start faster than raw downloads and match enterprise supply chain patterns.
* **Shorten names:** keep deployment names under ~30–40 characters to avoid Kubernetes 63-character limits.
* **New project storage option:** if “Existing cluster storage” is missing, create at least one Data Connection in that project first.

---

## Troubleshooting

### Known Issues (RHOAI 3.2)

### 1) “Request access to model catalog” after upgrade (2.24 → 3.2)

Fix by forcing a clean refresh:

```bash
oc delete configmap model-catalog-sources -n rhoai-model-registries
oc delete deployment model-catalog -n rhoai-model-registries
oc get pods -n rhoai-model-registries -l component=model-catalog -w
```

### 2) Silent failure due to name length > 63 characters

Workaround: shorten the name in the wizard (examples: `llama-3-8b-int8`, `mistral-small-24b-awq`).

### 3) Missing storage options in new projects

Workaround: create a Data Connection in the target project, then retry the wizard.

---

## Repository Structure

```
/
├── modules/                     # Antora course content (AsciiDoc)
│   ├── ROOT/pages/index.adoc     # Home (includes chapter content)
│   └── chapter1/pages/           # Course pages (intro, architecture, lab, troubleshooting)
│
├── antora.yml                    # Component descriptor
├── antora-playbook.yml           # Antora build playbook
└── README.md                     # This file
```

---

## Next Steps

* Treat catalog sources as code (GitOps) and standardize naming conventions.
* Align catalog deployments with Hardware Profiles to prevent “wrong GPU” deployments.
* Prefer ModelCars when available to reduce cold-start time and improve reliability.

---

## Additional Resources

* **Full course:** Antora course in this repository (see **Option 1: View the Full Course** above).
* **OpenShift AI:** [Red Hat OpenShift AI Documentation](https://access.redhat.com/documentation/en-us/red_hat_openshift_ai/).
