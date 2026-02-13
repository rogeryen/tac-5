# E2E Test: Generate Natural Language Query

Test the Generate Query button functionality in the Natural Language SQL Interface application.

## User Story

As a user
I want to generate example natural language queries based on my data
So that I can quickly explore my data without thinking of queries from scratch

## Test Steps

1. Navigate to the `Application URL`
2. Take a screenshot of the initial state
3. **Verify** the page title is "Natural Language SQL Interface"
4. **Verify** the "Generate Query" button is present next to "Upload Data" button
5. **Verify** the Generate Query button has the secondary-button style (white background, blue border)

6. Click the "Upload Data" button to open the modal
7. Click the "Users Data" sample button to upload sample data
8. **Verify** the modal closes and a success message appears
9. **Verify** the "users" table appears in the Available Tables section
10. Take a screenshot of the tables section

11. Click the "Generate Query" button
12. **Verify** the button shows a loading state (spinner)
13. Wait for the query to be generated
14. **Verify** the query input field is populated with a natural language query
15. **Verify** the generated query is non-empty
16. **Verify** the generated query is two sentences or less (count periods)
17. Take a screenshot of the generated query

18. Click the "Query" button to execute the generated query
19. **Verify** query results are displayed successfully
20. **Verify** the SQL translation is shown
21. **Verify** the results table contains data
22. Take a screenshot of the query results

23. Click the "Generate Query" button again
24. **Verify** the query input field is populated with a new query (overwrites the previous one)
25. **Verify** the new query is different from or could be the same as the first query (both are valid)
26. Take a screenshot of the second generated query

27. Test with no tables: Click the "×" button on the users table to remove it
28. **Verify** a confirmation dialog appears
29. Confirm the deletion
30. **Verify** the "No tables loaded" message appears
31. Click the "Generate Query" button
32. **Verify** an error message appears: "No tables available. Please upload data first."
33. Take a screenshot of the error state

## Success Criteria
- Generate Query button is present and styled correctly
- Button is positioned with Upload Data button with proper spacing
- Clicking button generates a natural language query
- Generated query populates the query input field (overwrites existing content)
- Generated queries are maximum two sentences long
- Generated query can be executed successfully
- Multiple clicks generate queries (can overwrite previous query)
- Error message shown when no tables exist
- Button shows loading state during generation
- 5 screenshots are taken
