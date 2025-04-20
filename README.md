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
blablabla

## Roadmap (for the last day)

- [ ] Expand the supported queries
    - [ ] Au niveau des patterns, il y en a peut être des redondants, fixer ça
- [ ] Clean up everything
- [ ] Add visualization