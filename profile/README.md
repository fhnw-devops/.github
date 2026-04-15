# FHNW DevOps – HS25 Group 10

Dieses Repository enthält alle Artefakte des **DevOps Foundations**-Moduls an der FHNW (HS25, Gruppe 10).  
Das Projekt besteht aus zwei Quarkus-Microservices, die sich gegenseitig Nachrichten über eine Orchestrierungsschicht austauschen, sowie der vollständigen CI/CD- und Kubernetes-Infrastruktur dafür.

---

## Übersicht der Module

fhnw-devops/

- [Connecting-Worlds](https://github.com/fhnw-devops/Connecting-Worlds)/ ← Orchestrierungs-Service (Balancer)
- [service-henz](https://github.com/fhnw-devops/service-henz)/ ← Chatbot-Microservice (OpenAI / LangChain4j)
- [gitlab-components](https://github.com/fhnw-devops/gitlab-components)/ ← Wiederverwendbare GitLab CI/CD-Templates
- [helm-resources](https://github.com/fhnw-devops/helm-resources)/ ← Kubernetes Helm Chart (Azure AKS)
- [k8s-resources](https://github.com/fhnw-devops/k8s-resources)/ ← Raw Kubernetes Manifeste
- [renovate](https://github.com/fhnw-devops/renovate)/ ← Renovate Bot Konfiguration

---

## [Connecting-Worlds](../Connecting-Worlds)

Der zentrale **Orchestrierungs-Service**, der mehrere Chatbots miteinander verbindet.  
Nachrichten werden per Round-Robin zwischen den registrierten Chatbot-Endpunkten weitergeleitet.  
Die ergebende Konversation ist über eine Qute-HTML-Seite einsehbar.

| Eigenschaft | Wert                                |
| ----------- | ----------------------------------- |
| Framework   | [Quarkus](https://quarkus.io/) 3.15 |
| Sprache     | Java 21                             |
| Trigger     | `@Scheduled` alle 30 Sekunden       |
| Health      | SmallRye Health (`/q/health`)       |
| Metriken    | Micrometer                          |

---

## [service-henz](../service-henz)

Ein eigenständiger **Chatbot-Microservice**, der Benutzerprompts über [LangChain4j](https://docs.langchain4j.dev/) an die **OpenAI API** weiterleitet.  
Exponiert einen REST-Endpunkt `GET /send?message=<text>`, den Connecting-Worlds aufruft.

| Eigenschaft     | Wert                           |
| --------------- | ------------------------------ |
| Framework       | [Quarkus](https://quarkus.io/) |
| Sprache         | Java 21                        |
| LLM-Integration | LangChain4j + OpenAI           |
| Endpunkt        | `GET /send?message=<prompt>`   |

---

## [gitlab-components](../gitlab-components)

Sammlung **wiederverwendbarer GitLab CI/CD-Templates**, die von den Service-Repositories per `include:` eingebunden werden.

---

## [helm-resources](../helm-resources)

**Kubernetes Helm Chart** für das Deployment aller Services auf **Azure AKS**.  
Unterstützt zwei Stages mit separaten Values-Dateien:

| Stage       | Values-Datei                                                         | Image-Tag        |
| ----------- | -------------------------------------------------------------------- | ---------------- |
| Development | [`values-dev.yaml`](../helm-resources/devops-helm/values-dev.yaml)   | `:main` + Digest |
| Production  | [`values-prod.yaml`](../helm-resources/devops-helm/values-prod.yaml) | Semver-Tag       |

---

## [k8s-resources](../k8s-resources)

**Raw Kubernetes Manifeste** (Alternative zu Helm) für das direkte Deployment per `kubectl apply`.  
Enthält je einen Ordner pro Service mit `deployment.yml`, `service.yml` und `config-map.yml`.

| Ordner                                                      | Inhalt                                      |
| ----------------------------------------------------------- | ------------------------------------------- |
| [`connecting-worlds/`](../k8s-resources/connecting-worlds/) | Manifeste für den Orchestrierungs-Service   |
| [`service-henz/`](../k8s-resources/service-henz/)           | Manifeste für den Chatbot-Service           |
| [`service-spuhler/`](../k8s-resources/service-spuhler/)     | Manifeste für den Chatbot-Service (Spuhler) |

---

## CI/CD-Ablauf

```
Code-Push
    │
    ▼
analysis ──► build ──► cve_check ──► release ──► release_push
  │             │          │
  │             │          └── Trivy / Grype (blockiert bei HIGH/CRITICAL)
  │             └────────────── Kaniko Docker-Build → GitLab Registry
  └──────────────────────────── Checkstyle, SBOM, CVE-Scan (Filesystem)
```

Pipelines werden **nicht** bei Git-Tags oder automatischen Changelog-Commits ausgeführt (verhindert Loops).

---
