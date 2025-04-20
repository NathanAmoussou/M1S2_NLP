# Finance SQL Database Interrogation in Natural Language

This project is more of an exercise prompted by NLP class of Professor E. Cabrio of Université Côte d'Azur. Here, we aim to provide a symbolic (and an attempted machine learning) approach to go from a natural language request to a properly parsed SQL request (tailored to our specific finance-oriented database).

## Manual (how to use)
blablabla

## Examples of supported queries
| ID  | Example query                                             | Informations         | Correctly supported |
| --- | --------------------------------------------------------- | -------------------- | ------------------- |
| 01  | Top 10 most valuable American IT companies                | Filtered Ranking     | No        |
| 02  | Show details for Apple Inc.                               | Company Details      | Yes       |
| 03  | Every French company valued over 1 B                      | Filtered List        | Yes       |
| 04  | List all sectors available                                | Metadata List        | Yes       |
| 05  | What is the market cap of Microsoft?                      | Single Value         | Yes       |
| 06  | Which German companies are in the Health Care sector?     | Filtered List        | Yes       |
| 07  | Find companies worth more than 500 billion dollars        | Filtered List        | Yes       |
| 08  | Show financials for Tesla                                 | Company Financials   | No        |
| 09  | List info tech companies in the US                        | Filtered List        | Yes       |
| 10  | Lowest 5 market cap companies in France                   | Filtered Ranking     | No        |
| 11  | Show country and sector for companies in Spain            | Company Attributes   | Yes       |
| 12  | What sector is Apple in?                                  | Single Value         | No        |
| 13  | Show the founding date for Microsoft                      | Single Value         | Yes       |
| 14  | List stock prices for IT companies                        | List of Values       | Yes       |
| 15  | Companies founded after 1990                              | Filtered List        | Yes       |
| 16  | Companies with stock price over \$500                     | Filtered List        | Yes       |
| 17  | USA companies in IT founded before 2000                   | Filtered List        | No        |
| 18  | List companies ordered by founding date                   | Ordered List         | Yes       |

A feature to add: Synonyms/Rephrasing, handle "based in" or "located in" for Country. Handle "worth" vs "market cap" vs "valuation".

## Known bugs & limitations

This NLP-to-SQL system demonstrates the core pipeline but has several known limitations that impact its accuracy and scope:

1.  **Intent Recognition:**
    *   Fails to consistently identify "Top N" / "Lowest N" ranking queries (e.g., "Top 10...", "Lowest 5..."), often defaulting to a standard list (`filter_list`) without applying limits or ordering.
    *   May fail to recognize specific single-column lookup queries (e.g., "What sector is Apple in?"), again defaulting to a broader list query.

2.  **Entity/Parameter Extraction:**
    *   **Monetary Values:** Cannot reliably parse short-form monetary values with units followed by punctuation (e.g., "1B."). These are often missed entirely or misclassified as simple numbers (`CARDINAL`), preventing the creation of corresponding threshold filters (e.g., `Marketcap > 1000000000`). Parses longer forms ("500 billion") correctly.
    *   **Date Filters:** While date comparisons ("founded after 1990") work in isolation, the system currently fails to apply them correctly when combined with other filters (e.g., country and sector) in the same query.
    *   **Entity Overrides:** Custom rules designed to identify specific entities (like single-word country names "France", "Spain") sometimes fail to override the default spaCy NER tags (e.g., `GPE`), although this is currently masked by the database scope limitation.

3.  **Normalization & Matching:**
    *   Relies on exact, case-insensitive matching after normalization against database terms. It lacks fuzzy matching capabilities and may fail if the query uses a slightly different name variant than stored in the database (e.g., "Apple" vs. "Apple Inc.").

4.  **Database Scope:**
    *   The underlying database (`companies_database.db`) currently contains data primarily for companies in the USA and India. While the parser correctly identifies terms for other countries (e.g., "French", "German"), it issues a warning and ignores them as filters because the corresponding data is not present. The generated SQL will therefore return results for all available countries in these cases.

5.  **Underlying Warnings:**
    *   Persistent spaCy warnings (`[W036] ... entity_ruler does not have any patterns defined.`) occur during setup. While core functionality seems largely intact, this suggests potential minor instability in how custom entity patterns are loaded or applied.

Addressing these limitations, particularly improving intent recognition and parsing complex numeric/date entities, would be key areas for future development.

## Roadmap (for the last day)

- [ ] Expand the supported queries
    - [ ] Au niveau des patterns, il y en a peut être des redondants, fixer ça
- [ ] Clean up everything
- [ ] Add visualization