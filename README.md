# [SQL] Count Occurrences Of Words In Drafts

Practice from [StrataScratch](https://platform.stratascratch.com/coding/9817-find-the-number-of-times-each-word-appears-in-drafts?code_type=3). 

- **Objective**: This query counts how many times each word appears in the `contents` field of the `google_file_store` table.
- **Practice Purpose**: Self-learning and reinforcement of SQL techniques such as data cleaning, aggregation, joins, subqueries, and window functions.
- **Outline**: 
    - **Practice** (practice problem and query output)
    - **Solution** (step-by-step explanation)
    - **Future Enhancements**


## Practice 

Find the number of times each word appears in the `contents` column across all rows in the `drafts` dataset. Output two columns: `word` and `occurrences`.

- **Table**: `google_file_store`

|   filename   |   contents   |
|--------------|--------------|
|  draft1.txt  |	The stock exchange predicts a bull market which would make many investors happy.|
|  draft2.txt  |	The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear market.|
|  final.txt   |	The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear market. As always predicting the future market is an uncertain game and all investors should follow their instincts and best practices.|


- **Expected Output**

|  word   | occurrences |
|---------|-------------|
|  market |	     6      |
|    a    |      5      |
|investors|	     4      |
|  and    |    	 4      |
|  the    |	     4      |


## Solution

_This section outlines my thought process for solving the problem._


Clean the text (remove punctuation)

Format it into a JSON array

Split using JSON_TABLE

Normalize (convert to lowercase)

Group and count the occurrences

Sort the result


### Final Syntax and Output using MySQL

***Syntax**

```sql
SELECT 
    LOWER(jt.word) AS word,
    COUNT(*) AS occurrences
FROM google_file_store g
JOIN JSON_TABLE(
    CONCAT(
        '["',
        REPLACE(REGEXP_REPLACE(g.contents, '[[:punct:]]', ''), ' ', '","'),
        '"]'
    ),
    '$[*]' COLUMNS (word VARCHAR(200) PATH '$')
) jt
GROUP BY LOWER(jt.word)
ORDER BY occurrences DESC;
```

***Output**

_Showing top 10 rows here._

|  word   | occurrences |
|---------|-------------|
|  market |	     6      |
|    a    |      5      |
|investors|	     4      |
|   and   |    	 4      |
|   the   |	     4      |
|   of	  |	     4      |
|   bull  |	     3      |
|  which  |	     3      |
|  would  |	     3      |
|  make   |	     3      |
