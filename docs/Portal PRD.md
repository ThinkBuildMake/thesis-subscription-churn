PRD

Thesis has no unified system for predicting which subscribers are at risk of canceling or for intervening before they churn. Churn signals currently sit fragmented across subscription data, email engagement, support interactions, and quiz responses in Snowflake with no predictive layer connecting them and zero automated response. This project builds an end-to-end system that scores churn risk, classifies the reason behind it, and autonomously takes action to retain the subscriber.

Problem & Context

Business Context

Subscription revenue is Thesis's economic foundation. Every subscriber lost requires a full replacement acquisition cost. Even a 2-3 percentage point reduction in monthly churn compounds into a significant shift in LTV and acquisition efficiency. Today, retention is entirely reactive. This project transforms it into a proactive, AI-driven function that operates continuously against the full subscriber base.

Impact Metrics

Churn rate reduction, model precision, model recall, intervention coverage, autonomous action rate, audit trail completeness

Requirements & Success Criteria

Functional Requirements

End-to-end pipeline: data ingestion, feature engineering, risk scoring, intervention selection, action execution, outcome logging. Explainable risk scores with top contributing factors per subscriber. Churn archetype classification (price sensitivity, efficacy dissatisfaction, subscription fatigue, life change) routing to distinct intervention paths. Minimum 4 intervention types: personalized check-in email, formulation adjustment suggestion, targeted retention offer, escalation to human. Configurable human-in-the-loop approval thresholds. Full action audit trail written to Snowflake. Graceful failure handling with state recovery. Production-ready at full subscriber base scale.

Performance Benchmarks

Model precision above 70% in the top risk decile against historical churn data. Model recall above 60% for pre-cancellation churn detection. Intervention coverage above 85% of identified at-risk subscribers. Autonomous action rate above 70% without human approval. Scoring pipeline must process the entire active subscriber base on a scheduled or continuous basis. 100% of interventions logged with full context to Snowflake.

Code Quality Expectations

Modular architecture with clear separation between prediction pipeline, agent logic, and intervention execution. Comprehensive documentation sufficient for a Thesis engineer to maintain and extend from day one. Automated testing for critical paths (scoring accuracy, intervention routing, failure recovery). No raw customer PII in logs or training data. Clean error handling with structured logging. Code organized for extensibility to new data sources and intervention types.

Time Constraints

3-5 days

Technical Contact

Yes. Raphael Cattan (raphael@takethesis.com), Director of Data Analytics, will be available for questions about the data environment, Snowflake schema, and project scope.

Technology

Required Languages

Python, SQL

AI / ML Frameworks

LLM APIs (Anthropic Claude or OpenAI), scikit-learn or equivalent for churn prediction modeling, LangChain/LangGraph or equivalent for agent orchestration

Dev Tools

Git, Docker, FastAPI, Snowflake connector (snowflake-connector-python or snowpark), testing frameworks (pytest)

Cloud Platforms

Snowflake (primary data platform). Cloud deployment platform TBD during onboarding.

Other Requirements

All data reads from existing Snowflake views covering subscriptions, orders, email engagement (Klaviyo sync), support interactions (Zendesk sync), and quiz data (7.3M+ data points). All outputs (risk scores, intervention logs, outcome tracking) must be written back to Snowflake and be queryable in Metabase. Direct API connections to source systems (Shopify, Klaviyo, Zendesk) should only be used when real-time data is required beyond what the warehouse provides. The system must gracefully handle subscribers with incomplete data (no quiz responses, limited engagement history). Human-in-the-loop thresholds must be configurable without code changes.

Off-Limits Tech

Do not build direct API connectors to source systems (Shopify, Klaviyo, Zendesk) when the data already exists in Snowflake. Do not include raw customer PII in logs or training data. Do not build a static rules engine disguised as an agent ("if score > 0.7, send email" is not intelligent intervention). Do not deliver a dashboard or report as the output; the deliverable is autonomous action. Do not fine-tune models where simpler approaches suffice.

Submission & AI Policy

AI Usage Documentation

Required

Required Deliverables

    Source Code
    Technical Documentation
    Demo Video
    Deployment Guide
    Test Results
    AI Usage Log