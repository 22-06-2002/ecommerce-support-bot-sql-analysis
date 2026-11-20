# ecommerce-support-bot-sql-analysis
Built a support agent (Botpress) handling order status, returns, refunds and ticket escalation in English, Telugu and Hindi. Tested with 50+ conversations including adversarial cases; analyzed failure patterns by intent and language using SQL (CTEs, window functions).
-- 1. Overall accuracy
SELECT ROUND(100.0 * SUM(CASE WHEN correct='Y' THEN 1 ELSE 0 END) / COUNT(*), 1) AS accuracy_pct
FROM conversations;

-- 2. Volume and accuracy by intent
SELECT intent, COUNT(*) AS total,
       ROUND(100.0 * SUM(CASE WHEN correct='Y' THEN 1 ELSE 0 END) / COUNT(*), 1) AS accuracy_pct
FROM conversations
GROUP BY intent
ORDER BY total DESC;

-- 3. Top failure types
SELECT failure_type, COUNT(*) AS total
FROM conversations
WHERE correct='N'
GROUP BY failure_type
ORDER BY total DESC;

-- 4. Accuracy by language
SELECT language, COUNT(*) AS total,
       ROUND(100.0 * SUM(CASE WHEN correct='Y' THEN 1 ELSE 0 END) / COUNT(*), 1) AS accuracy_pct
FROM conversations
GROUP BY language;

-- 5. Bot resolution rate vs tickets
SELECT ROUND(100.0 * SUM(CASE WHEN resolved_by_bot='Y' THEN 1 ELSE 0 END) / COUNT(*), 1) AS resolved_pct,
       ROUND(100.0 * SUM(CASE WHEN ticket_created='Y' THEN 1 ELSE 0 END) / COUNT(*), 1) AS ticket_pct
FROM conversations;

-- 6. Intents causing the most tickets (window function)
WITH t AS (
  SELECT intent, COUNT(*) AS tickets FROM conversations WHERE ticket_created='Y' GROUP BY intent
)
SELECT intent, tickets, RANK() OVER (ORDER BY tickets DESC) AS rnk FROM t;

-- 7. Busiest hours
SELECT SUBSTR(time, 1, 2) AS hour, COUNT(*) AS chats
FROM conversations
GROUP BY hour
ORDER BY chats DESC;

-- 8. Missed escalations
SELECT COUNT(*) AS missed
FROM conversations
WHERE intent='complaint' AND failure_type='escalation_missed';
