# Finance SQL Database Interrogation in Natural Language

This project is more of an exercise prompted by NLP class of Professor E. Cabrio of Université Côte d'Azur. Here, we aim to provide a symbolic (and an attempted machine learning) approach to go from a natural language request to a properly parsed SQL request (tailored to our specific finance-oriented database).

## Manual (how to use)
blablabla

## Examples of supported queries
### Already supported (04/20/2025)
|**Example query** | **Informations** |
| ------------- | ------------ |
| "Top 10 most valuable American IT companies." | Filtered Ranking |
| "Show details for Apple Inc." | Company Info Lookup (using specific name from DB potential) |
| "Every French company valued over 1B.""Every French company valued over 1B." | Threshold Filtering |
| "List all sectors available." | A potential simpler query |
| "What is the market cap of Microsoft?" | Specific detail lookup |

### To be supported soon
|**Example query** | **Informations** |
| ------------- | ------------ |
| "What sector is Apple in?" | Specific Column Requests |
| "Show the founding date for Microsoft." |  Specific Column Requests |
| "List stock prices for IT companies." | Specific Column Requests |
| "Companies founded after 1990." | Filtering on Different Columns |
| "Companies with stock price over $500." | Filtering on Different Columns (Needs numeric parsing for price) |
| "USA companies in IT founded before 2000." | Multiple Conditions (Combines Country, Sector, Founded) |
| "Average market cap of companies in India." | Simple Aggregations |
| "Count companies in the energy sector." | Simple Aggregations |
| "List companies ordered by founding date." | Ordering by other columns |

A feature to add: Synonyms/Rephrasing, handle "based in" or "located in" for Country. Handle "worth" vs "market cap" vs "valuation".

## Known bugs & limitations
blablabla

## Roadmap (for the last day)

- [ ] Expand the supported queries
    - [ ] Au niveau des patterns, il y en a peut être des redondants, fixer ça
- [ ] Clean up everything
- [ ] Add visualization