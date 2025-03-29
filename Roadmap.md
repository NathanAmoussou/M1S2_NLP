## **Refined Roadmap: NLP-to-SQL Engine for Finance Queries (v2)**

This roadmap aligns with the SQLite database (`companies_database.db`) populated in the previous steps and incorporates recommendations for more robust NLP techniques.

---

## **Phase 1: Project Setup & Scope (Partially Complete)**

### **Step 1: Define Scope and Query Types (Review & Confirm)**

The NLP engine aims to handle queries against the populated `companies` table.

*   **Query Types:**
    1.  **Company Info Lookup**:
        *   _"Show details for Apple."_
        *   → `SELECT * FROM companies WHERE Security='Apple Inc.';` *(Note: Use `Security` column, potentially add fuzzy matching later)*
    2.  **Filtered Ranking**:
        *   _"Top 10 most valuable American IT companies."_
        *   → `SELECT Security, Marketcap FROM companies WHERE Country='USA' AND Sector='Information Technology' ORDER BY Marketcap DESC LIMIT 10;` *(Using `Marketcap` for "valuable", `Country`, `Sector`)*
    3.  **Threshold Filtering**:
        *   _"Every French company valued over 1B."_
        *   → `SELECT Security FROM companies WHERE Country='France' AND Marketcap > 1000000000;` *(Using `Marketcap` and handling numeric values)*
    4.  **Comparisons & Trends (Requires Additional Data)**:
        *   _"Which company grew the most last year?"_
        *   → **Requires** columns like `Marketcap_PreviousYear`. *Marked as out-of-scope unless data is added.*

*   **Language:** English (for consistency with data and common NLP models).

---

## **Phase 2: Database Foundation (Complete)**

### **Step 2: Database Schema Definition (Actual Schema)**

The schema reflects the `companies_database.db` created previously.

*   **Table:** `companies`
*   **Columns:**
    *   `Symbol` (TEXT, Primary Key or Unique Identifier)
    *   `Security` (TEXT, Company Name)
    *   `Sector` (TEXT)
    *   `Industry` (TEXT)
    *   `Founded` (TEXT or INTEGER)
    *   `Marketcap` (REAL or INTEGER, Represents company value)
    *   `Stockprice` (REAL)
    *   `Country` (TEXT)

*(Self-Correction: Removed `id` as `Symbol` is likely the unique key. Adjusted types based on cleaned data - `Marketcap`/`Stockprice` are numeric).*

```sql
-- This reflects the structure created by the Python script
-- Example representation (Actual types determined by pandas/SQLite)
CREATE TABLE companies (
    Symbol TEXT UNIQUE NOT NULL,
    Security TEXT,
    Sector TEXT,
    Industry TEXT,
    Founded TEXT, -- Or INTEGER if cleaned
    Marketcap REAL, -- Or INTEGER/BIGINT
    Stockprice REAL,
    Country TEXT
);
```

### **Step 3: Populate the Database (DONE)**

*   **Method:** Completed using Python (`pandas`, `sqlite3`) to merge and load data from CSV files into `companies_database.db`.
*   **Note:** For trend analysis queries (e.g., growth), additional data sources providing historical market cap or financials would need to be integrated in a similar fashion.

---

## **Phase 3: NLP Processing Pipeline**

### **Step 4: Core NLP Processing (Tokenization, POS, Lemmatization)**

*   **Goal:** Break down queries and understand basic word roles.
*   **Tool:** **spaCy** is highly recommended.
    ```python
    import spacy
    # Load a suitable spaCy model (small, medium, or large)
    nlp = spacy.load("en_core_web_sm") # Or medium/large for better accuracy

    query = "Top 10 most valuable American IT companies"
    doc = nlp(query)

    print("Token | Lemma | POS | Dependency")
    for token in doc:
        print(f"{token.text} | {token.lemma_} | {token.pos_} | {token.dep_}")
    ```

### **Step 5: Enhanced Named Entity Recognition (NER) & Term Extraction**

*   **Goal:** Identify key pieces of information (Companies, Locations, Sectors, Numbers, Financial terms).
*   **Tools & Techniques:**
    1.  **spaCy Base NER:** Identify standard entities (ORG, GPE, CARDINAL).
    2.  **Gazetteers (Lists):** Create lists of known entities *from your database* (e.g., unique `Security` names, `Sector` values, `Country` names).
    3.  **spaCy `PhraseMatcher`:** Efficiently find terms from your gazetteers within the query.
    4.  **spaCy `EntityRuler`:** Add custom rules to identify patterns (e.g., monetary values like "$1B", "over 500 million", specific financial indicators if added later).
    ```python
    # Example using PhraseMatcher with data from DB
    # Assume unique_sectors = ['Information Technology', 'Health Care', ...] fetched from DB
    from spacy.matcher import PhraseMatcher
    matcher = PhraseMatcher(nlp.vocab, attr='LOWER') # Case-insensitive matching
    sector_patterns = [nlp.make_doc(sector) for sector in unique_sectors]
    matcher.add("SECTOR_TERM", sector_patterns)

    doc = nlp("List info tech companies in the USA")
    matches = matcher(doc)
    for match_id, start, end in matches:
        span = doc[start:end]
        print(f"Found Sector: {span.text}") # Output: Found Sector: info tech

    # Combine with base NER
    for ent in doc.ents:
        print(f"Found Base Entity: {ent.text} ({ent.label_})") # Output: Found Base Entity: USA (GPE)
    ```

### **Step 6: Query Structure Parsing (Intent & Parameter Extraction)**

*   **Goal:** Determine the user's intent (e.g., find top N, filter, lookup) and extract all relevant parameters and constraints identified in Step 5. Avoid brittle CFGs.
*   **Tool:** **spaCy's `Matcher`** based on token attributes (lemma, POS, dependency relations, entity types) or custom rule-based logic in Python.
*   **Output:** A structured representation (e.g., a Python dictionary).

    ```python
    # Conceptual Example (Logic to be implemented)
    def parse_query_structure(doc, identified_entities):
        parsed_structure = {'intent': None, 'select_cols': ['Security', 'Marketcap'], 'filters': [], 'limit': None, 'order_by': None}

        # --- Rule examples using token analysis & identified_entities ---
        # Rule 1: Detect "Top N" intent
        if any(tok.lemma_ in ["top", "highest", "largest"] for tok in doc) and \
           any(ent['label'] == 'CARDINAL' for ent in identified_entities):
             parsed_structure['intent'] = 'find_top'
             parsed_structure['limit'] = next(ent['text'] for ent in identified_entities if ent['label'] == 'CARDINAL')
             # Default order for 'top' queries
             parsed_structure['order_by'] = {'column': 'Marketcap', 'direction': 'DESC'}

        # Rule 2: Detect filtering criteria
        for ent in identified_entities:
            if ent['label'] == 'GPE': # Country
                parsed_structure['filters'].append({'column': 'Country', 'value': ent['text']})
            elif ent['label'] == 'SECTOR_TERM': # Custom entity from PhraseMatcher
                parsed_structure['filters'].append({'column': 'Sector', 'value': ent['text']})
            elif ent['label'] == 'MONEY' or ent['label'] == 'VALUE_COMPARISON': # Custom entity for "> 1B"
                 # Add logic to extract operator and value
                 # parsed_structure['filters'].append({'column': 'Marketcap', 'operator': '>', 'value': 1000000000})
                 pass # Placeholder for value comparison logic

        # Rule 3: Detect specific company lookup
        if any(ent['label'] == 'ORG' or ent['label'] == 'COMPANY_NAME' for ent in identified_entities):
             # Check if other rules conflict (e.g. not a "top N" query)
             if parsed_structure['intent'] is None:
                  parsed_structure['intent'] = 'lookup_details'
                  parsed_structure['select_cols'] = ['*'] # Select all columns for details
                  # Add filter for the specific company name
                  parsed_structure['filters'].append({'column': 'Security', 'value': ent['text']}) # Might need fuzzy matching

        # Add more rules for different query structures...

        # Refine select_cols based on keywords (e.g., "show valuation", "list names")
        # ...

        return parsed_structure

    # Example Usage (assuming identified_entities is populated from Step 5)
    # identified_entities = [{'text': '10', 'label': 'CARDINAL'}, {'text': 'American', 'label': 'NORP'}, {'text': 'IT', 'label': 'SECTOR_TERM'}] # Simplified example
    # parsed = parse_query_structure(doc, identified_entities)
    # print(parsed)
    ```

---

## **Phase 4: SQL Generation**

### **Step 7: Convert Parsed Structure to SQL Query**

*   **Goal:** Translate the structured dictionary from Step 6 into a valid SQL query for the `companies_database.db`.
*   **Method:** Implement a Python function that constructs the SQL string based on the parsed intent and parameters.

    ```python
    def generate_sql_from_structure(parsed_structure):
        if not parsed_structure or parsed_structure['intent'] is None:
            return "-- Invalid or unparsed query"

        select_clause = f"SELECT {', '.join(parsed_structure['select_cols'])}"
        from_clause = "FROM companies"
        where_clause = ""
        orderby_clause = ""
        limit_clause = ""

        # Build WHERE clause
        if parsed_structure['filters']:
            conditions = []
            for f in parsed_structure['filters']:
                # Basic equality - needs enhancement for operators (> < etc) and type handling (quotes for strings)
                operator = f.get('operator', '=')
                value = f['value']
                # Basic quoting for strings, assume numeric otherwise (needs refinement)
                sql_value = f"'{value}'" if isinstance(value, str) else str(value)
                conditions.append(f"{f['column']} {operator} {sql_value}")
            where_clause = "WHERE " + " AND ".join(conditions)

        # Build ORDER BY clause
        if parsed_structure['order_by']:
            ob = parsed_structure['order_by']
            orderby_clause = f"ORDER BY {ob['column']} {ob['direction']}"

        # Build LIMIT clause
        if parsed_structure['limit']:
            limit_clause = f"LIMIT {parsed_structure['limit']}"

        # Combine clauses
        query_parts = [select_clause, from_clause, where_clause, orderby_clause, limit_clause]
        sql_query = " ".join(filter(None, query_parts)).strip() + ";" # Filter removes empty strings

        return sql_query

    # Example Test (using conceptual output from Step 6)
    # parsed_query1 = {
    #     'intent': 'find_top',
    #     'select_cols': ['Security', 'Marketcap'],
    #     'filters': [{'column': 'Country', 'value': 'USA'}, {'column': 'Sector', 'value': 'Information Technology'}],
    #     'limit': 10,
    #     'order_by': {'column': 'Marketcap', 'direction': 'DESC'}
    # }
    # print(generate_sql_from_structure(parsed_query1))
    # Expected: SELECT Security, Marketcap FROM companies WHERE Country = 'USA' AND Sector = 'Information Technology' ORDER BY Marketcap DESC LIMIT 10;
    ```

### **Step 8: Test Rule-Based SQL Generation**

*   **Goal:** Verify the end-to-end process for defined query types.
*   **Method:** Create test cases with natural language queries, run them through Steps 4-7, and compare the generated SQL to manually crafted expected SQL. Execute the generated SQL against the database to ensure it runs correctly.

---

## **Phase 5: Machine Learning Enhancement (Optional / Future)**

### **Step 9: Intent Classification / Slot Filling**

*   **Goal:** Improve robustness or handle more complex queries by training ML models.
    *   **Intent Classifier:** Predict the query type (lookup, filter, rank) using models like Naive Bayes, SVM, or Transformers (with libraries like Scikit-learn or Hugging Face).
    *   **Slot Filling:** Use sequence labeling models (like BiLSTM-CRF or Transformer-based NER) to extract parameters (company name, location, criteria) more dynamically.
*   **Note:** This adds significant complexity and requires labeled training data. Recommended after the rule-based system is functional.

---

## **Phase 6: Testing & Evaluation**

### **Step 10: Comprehensive Validation**

*   **Goal:** Evaluate the accuracy and robustness of the NLP-to-SQL system.
*   **Metrics:**
    *   **Parsing Accuracy:** Did the system correctly identify intent and parameters?
    *   **SQL Correctness:** Is the generated SQL syntactically valid?
    *   **Execution Accuracy:** Does the generated SQL produce the correct results when run against the database?
*   **Method:** Create a diverse test set of queries. Compare generated SQL vs. expected SQL. Run generated SQL and compare results. For ML (Step 9), use standard classification/NER metrics (Accuracy, Precision, Recall, F1).

---

## **Phase 7: Deployment (Optional / Future)**

### **Step 11: Build API Interface**

*   **Goal:** Expose the NLP-to-SQL functionality via an API.
*   **Tool:** **Flask** or FastAPI are good choices.
    ```python
    # Flask Example Snippet (requires implementing the full pipeline)
    from flask import Flask, request, jsonify
    # Assume full_pipeline(query) performs steps 4-7
    # from your_nlp_module import full_pipeline

    app = Flask(__name__)

    @app.route('/query_to_sql', methods=['POST'])
    def translate_query():
        data = request.json
        human_query = data.get("query")
        if not human_query:
            return jsonify({"error": "No query provided"}), 400

        try:
            # Replace full_pipeline with your actual function integrating steps 4-7
            # sql_query = full_pipeline(human_query)
            # Placeholder:
            doc = nlp(human_query)
            # entities = extract_entities(doc) # Placeholder for Step 5 logic
            # parsed = parse_query_structure(doc, entities) # Placeholder for Step 6 logic
            # sql_query = generate_sql_from_structure(parsed) # Placeholder for Step 7 logic
            sql_query = "-- Pipeline implementation needed" # Replace with actual call
            return jsonify({"human_query": human_query, "generated_sql": sql_query})
        except Exception as e:
            # Log the error internally
            print(f"Error processing query: {e}")
            return jsonify({"error": "Failed to process query"}), 500

    # if __name__ == '__main__':
    #     app.run(debug=True) # Add host='0.0.0.0' for accessibility if needed
    ```

### **Step 12: Deploy the API**

*   **Goal:** Make the service available.
*   **Options:** Docker container, Cloud platforms (Heroku, AWS Elastic Beanstalk, Google Cloud Run), Serverless functions (AWS Lambda, Google Cloud Functions).

---

## **Final Summary (Refactored)**

| **Phase**                 | **Steps**                                           | **Status / Focus**                       |
| :------------------------ | :-------------------------------------------------- | :--------------------------------------- |
| **Setup & Scope**       | 1. Define Query Types                             | Review SQL examples, Confirm Scope       |
| **Database Foundation**   | 2. Define Schema, 3. Populate DB                  | **DONE**                                 |
| **NLP Processing**        | 4. Core NLP, 5. NER & Term Extraction, 6. Parsing | **NEXT MAJOR FOCUS** - Implement SpaCy |
| **SQL Generation**        | 7. Convert Structure->SQL, 8. Test Generation   | Implement mapping, Test end-to-end     |
| **ML Enhancement (Opt.)** | 9. Intent/Slot Models                             | Future consideration                     |
| **Testing & Eval**      | 10. Comprehensive Validation                      | Crucial after Phase 4                  |
| **Deployment (Opt.)**     | 11. Build API, 12. Deploy                         | Future consideration                     |

---

This refactored roadmap should now be accurately aligned with your populated database and provides more concrete suggestions for the NLP implementation using spaCy's features. The next logical step is to start implementing Phase 3 (Steps 4, 5, and 6).