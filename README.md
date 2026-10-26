\# M01 Clinical Agent: Orchestration \& HITL Scheduling Suite



Arquitectura conversacional para centros de salud orientada a resolver la fricción operativa en agendamiento clínico, consulta de disponibilidad en tiempo real y captura estructurada de prospectos mediante modelos de lenguaje (LLMs) asistidos por humanos (\*\*Human-in-the-Loop\*\*).



> \*\*Aviso de Propiedad Intelectual:\*\* Este repositorio constituye un \*Case Study / Technical Showcase\*. El código de producción y los adaptadores propietarios se mantienen en repositorio privado bajo acuerdos de confidencialidad clínica.



\---



\## 🎯 Problema de Negocio y Enfoque



Los centros médicos sufren altos costos de atención telefónica, fuga de prospectos y errores en agendamiento. La automatización 100% autónoma mediante LLMs suele fallar en entornos clínicos debido a alucinaciones de turnos, normativas de datos médicos y reglas de negocio complejas (duraciones por prestación, especialidades médicas).



\*\*M01-Agent\*\* desacopla la inteligencia conversacional de la lógica de negocio clínica:

\- \*\*Enrutamiento adaptativo multi-software:\*\* Abstracción unificada para SACMED, Medilink, Dentalink y Reservo.

\- \*\*Validación determinista:\*\* Algoritmo local Módulo 11 para RUT chileno y control estricto de zona horaria (`America/Santiago`).

\- \*\*Arquitectura HITL (Human-in-the-Loop):\*\* El asistente pre-califica, asocia prestaciones oficiales SAP/SACMED y pre-reserva, derivando la confirmación definitiva al personal administrativo.



\---



\## 🏗️ Diagrama de Arquitectura del Sistema



```mermaid

flowchart TD

&#x20;   subgraph Canales \[Canales de Entrada]

&#x20;       Web\[Web Console / React 18]

&#x20;       Wsp\[WhatsApp Business Webhook]

&#x20;   end



&#x20;   subgraph Gateway \[FastAPI Gateway \& Seguridad]

&#x20;       Guard\[Input Guardrails \& Sanitizer<br/>Regex / Anti-Prompt Injection]

&#x20;       Rate\[Rate Limiting \& Session State]

&#x20;       Auth\[HMAC X-Admin-API-Key Validator]

&#x20;   end



&#x20;   subgraph LLM\_Layer \[Capa de Inteligencia Conversacional]

&#x20;       LLM\[AsyncOpenAI Orchestrator]

&#x20;       PromptEngine\[Dynamic System Prompt Builder<br/>Identidad + Género + Catálogo SAP]

&#x20;       Tools\[Function Calling Engine]

&#x20;   end



&#x20;   subgraph Clinical\_Core \[Capa de Adaptadores Clínicos]

&#x20;       Router\[Clinical Multi-Software Router]

&#x20;       ValRUT\[RUT Modulo 11 Determinista]

&#x20;       Lock\[Soft-Lock 20 min Transaction Window]

&#x20;       Sem\[asyncio.Semaphore Concurrency Control]

&#x20;   end



&#x20;   subgraph Integraciones \[Sistemas de Salud Integrados]

&#x20;       SACMED\[SACMED / SAP Connector]

&#x20;       Medilink\[Medilink Adaptor]

&#x20;       Reservo\[Reservo Adaptor]

&#x20;       Dentalink\[Dentalink Adaptor]

&#x20;   end



&#x20;   subgraph Persistencia \[Persistencia \& Calidad]

&#x20;       PG\[(PostgreSQL 16 Pool Anti-Leaks)]

&#x20;       Memoria\[Memoria Conversacional Contextual]

&#x20;       Leads\[Tracking de Prospectos \& Citas HITL]

&#x20;       QGate\[Quality Gate Automatizado<br/>check\_calidad.ps1]

&#x20;   end



&#x20;   Canales --> Guard

&#x20;   Guard --> Rate

&#x20;   Rate --> Auth

&#x20;   Auth --> LLM

&#x20;   PromptEngine --> LLM

&#x20;   LLM --> Tools



&#x20;   Tools --> Router

&#x20;   Router --> ValRUT

&#x20;   ValRUT --> Lock

&#x20;   Lock --> Sem



&#x20;   Sem --> SACMED

&#x20;   Sem --> Medilink

&#x20;   Sem --> Reservo

&#x20;   Sem --> Dentalink



&#x20;   LLM <--> PG

&#x20;   Router <--> PG

&#x20;   PG --- Memoria

&#x20;   PG --- Leads

&#x20;   QGate -.-> Gateway

&#x20;   QGate -.-> Clinical\_Core

