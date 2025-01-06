# iGaming Analytics: Player Behavior, Revenue, and Trends

## Project Overview

This project delves into iGaming analytics using SQL and Power BI, focusing on player behavior, revenue generation, and industry trends. It aims to derive actionable insights such as game preferences, retention patterns, and revenue forecasting, while showcasing advanced data analysis and visualization skills.

## Dataset Description

### Database Schema

-   **Players Table**: Stores player demographic information.
-   **Bets Table**: Tracks betting activity.
-   **BetType Table**: Contains types of bets placed.
-   **Platforms Table**: Represents different platforms where bets are placed.
-   **Games Table**: Details games available for betting.
-   **GameProviders Table**: Lists game providers.
-   **GameType Table**: Classifies games by type.
-   **Sessions Table**: Tracks player sessions.
-   **SessionType Table**: Categorizes session types.
-   **Promotions Table**: Details promotions offered to players.
-   **PromotionRedemption Table**: Tracks redemption of promotions.
-   **Transactions Table**: Logs financial transactions.
-   **TransactionType Table**: Defines transaction types.
-   **TransactionStatus Table**: Represents statuses of transactions.
-   **PaymentMethods Table**: Lists payment methods used.
-   **VIPStatus Table**: Represents VIP levels of players.
-   **PlayerStatus Table**: Tracks player activity status.

## Phase 1: Data Cleaning and Validation

### Missing Value Analysis

**Objective:** Identify and handle missing values in the dataset.

**Query:**

```sql
	SELECT
		t.TABLE_NAME,
		COLUMN_NAME,
		COUNT(*) AS MissingCount
	FROM INFORMATION_SCHEMA.COLUMNS c
	JOIN INFORMATION_SCHEMA.TABLES t ON c.TABLE_NAME = t.TABLE_NAME
	WHERE
		c.IS_NULLABLE = 'YES'
		AND t.TABLE_TYPE = 'BASE TABLE'
	GROUP BY
		t.TABLE_NAME,
		COLUMN_NAME;
```

**Result:**

-   No Missing values in nullabl columns /columns that allow NULL values/.

### Duplicate Records

**Objective:** Identify and remove duplicate records.

**Query 1: Identify duplicate records in PLAYERS table**

```sql
		SELECT
			PlayerID,
			COUNT(*) AS DuplicateCount
		FROM Players
		GROUP BY PlayerID
		HAVING COUNT(*) > 1;
```

**Result:**

-   No duplicate records.

**Query 2: Identify duplicate records in BETS table**

```sql
		SELECT
			PlayerID,
			BetID,
			COUNT(*) AS DuplicateCount
		FROM Bets
		GROUP BY PlayerID, BetID
		HAVING COUNT(*) > 1;
```

**Result:**

-   No duplicate records.

**Query 3: Identify duplicate records in Sessions table**

```sql
		SELECT
			PlayerID,
			SessionID,
			COUNT(*) AS DuplicateCount
		FROM Sessions
		GROUP BY PlayerID, SessionID
		HAVING COUNT(*) > 1;
```

**Result:**

-   No duplicate records.

**Query 4: Identify duplicate records in Transactions table**

```sql
		SELECT
			PlayerID,
			TransactionID,
			COUNT(*) AS DuplicateCount
		FROM Transactions
		GROUP BY PlayerID, TransactionID
		HAVING COUNT(*) > 1;
```

**Result:**

-   No duplicate records.

### Date and Time Validation

**Objective:** Ensure consistency and accuracy of date and time data.

**Query 1: StartTime should be before EndTime in SESSIONS**

```sql
	SELECT
		SessionID,
		StartTime,
		EndTime
	FROM Sessions
	WHERE StartTime >= EndTime;
```

**Result:**

-   No EndTime is before StartTime.

**Query 2: RegistrationDate should not exceed the current date in PLAYERS table**

```sql
	SELECT
		PlayerID,
		RegistrationDate
	FROM Players
	WHERE RegistrationDate > GETDATE();
```

**Result:**

-   No RegistrationDate is after current daee.

**Query 3: BetDate should not exceed the current date in BETS table**

```sql
	SELECT
		PlayerID,
		BetDate
	FROM Bets
	WHERE BetDate > GETDATE();
```

**Result:**

-   No BetDate is after current date.

### Foreign Key Integrity Check

**Objective:** Ensure all foreign key relationships are valid.

**Example Query: Example to check BETS → PLAYERS relationship**

```sql
	SELECT
		b.BetID,
		b.PlayerID
	FROM Bets b
	LEFT JOIN Players p ON b.PlayerID = p.PlayerID
	WHERE p.PlayerID IS NULL;
```

**Result:**

-   Series of queries example query are conducted. All foreign key relationships are valid.

## Phase 2: Data Analysis (SQL)

**Objective:** Derive insights from the cleaned dataset using SQL queries.

### Gross Gaming Revenue (GGR) Analysis

**Objective 1:** Calculate total GGR by platform.

**Query 1:**

```sql
	SELECT
		p.PlatformName,
		SUM(b.BetAmount - b.WinAmount) AS TotalGGR
	FROM Bets b
	JOIN Platforms p ON b.PlatformID = p.PlatformID
	GROUP BY p.PlatformName
	ORDER BY TotalGGR DESC

```

**Result:**

-   The table below shows the GGR calculated for each platform, ranked in descending order:

| Platform Name | Total GGR (USD) |
| ------------- | --------------- |
| Desktop       | 1,793,047.91    |
| Android App   | 1,760,828.94    |
| iOS App       | 1,753,705.24    |
| Tablet        | 1,686,770.94    |
| Mobile Web    | 1,684,954.24    |

**Insights:**

-   Desktop is the top-performing platform with the highest GGR.
-   Android and iOS apps closely follow, indicating significant revenue from mobile applications.
-   Tablet and Mobile Web platforms contribute less GGR compared to other platforms but are still considerable.

### Gross Gaming Revenue (GGR) Analysis

**Objective 2:** Calculate total GGR by Game Type.

**Query 2:**

```sql
	SELECT
		gt.TypeName,
		SUM(b.BetAmount - b.WinAmount) AS TotalGGR
	FROM Bets b
	JOIN Games g ON b.GameID = g.GameID
	JOIN GameType gt ON g.GameTypeID = gt.GameTypeID
	GROUP BY gt.TypeName
	ORDER BY TotalGGR DESC
```

**Result:**

-   The table below shows the GGR calculated for each Game Type, ranked in descending order:

| Game Type      | Total GGR (USD) |
| -------------- | --------------- |
| Live Casino    | 3,071,657.78    |
| Table Games    | 2,444,372.67    |
| Poker          | 1,824,493.59    |
| Sports Betting | 1,503,337.54    |
| Slots          | -164,554.31     |

**Insights:**

-   Live Casino is the top-performing game type
-   Slots category has negative GGR and requires high attention

**Objective 3:** Calculate total GGR by Players VIP Status.

**Query 3:**

```sql
	SELECT
		v.StatusName,
		SUM(b.BetAmount - b.WinAmount) AS TotalGGR
	FROM Bets b
	JOIN Players p ON b.PlayerID = p.PlayerID
	JOIN VIPStatus v ON p.VIPStatusID = v.VIPStatusID
	GROUP BY v.StatusName
	ORDER BY TotalGGR DESC
```

**Result:**

-   The table below shows the GGR calculated for each VIP status of Players, ranked in descending order:

| StatusName | Total GGR (USD) |
| ---------- | --------------- |
| Silver     | 2,604,307.23    |
| Gold       | 2,601,735.56    |
| Bronze     | 2,594,058.42    |
| Platinum   | 442,456.14      |
| Diamond    | 436,749.92      |

**Insights:**

-   Players with Silver Status has the highest Total GGR
-   GGR of Players with Silver, Gold and Bronze statuses are very close
-   Players with Diamont and Platinum statuses are generatig lowest total Revenue

### Player Segmentation Analysis

**Objective:** Segment players based on age and spending habits to identify key player groups.

**Query:**

```sql
	SELECT
		CASE
			WHEN DATEDIFF(YEAR, p.DateOfBirth, GETDATE()) < 25 THEN '18-24'
			WHEN DATEDIFF(YEAR, p.DateOfBirth, GETDATE()) BETWEEN 25 AND 34 THEN '25-34'
			WHEN DATEDIFF(YEAR, p.DateOfBirth, GETDATE()) BETWEEN 35 AND 44 THEN '35-44'
			ELSE '45+'
		END AS AgeGroup,
		COUNT(DISTINCT p.PlayerID) AS PlayerCount,
		SUM(b.BetAmount) AS TotalBetAmount
	FROM Players p
	LEFT JOIN Bets b ON p.PlayerID = b.PlayerID
	GROUP BY
		CASE
			WHEN DATEDIFF(YEAR, p.DateOfBirth, GETDATE()) < 25 THEN '18-24'
			WHEN DATEDIFF(YEAR, p.DateOfBirth, GETDATE()) BETWEEN 25 AND 34 THEN '25-34'
			WHEN DATEDIFF(YEAR, p.DateOfBirth, GETDATE()) BETWEEN 35 AND 44 THEN '35-44'
			ELSE '45+'
		END
	ORDER BY TotalBetAmount DESC;
```

**Result:**

-   The table bellow shows players count and total bets for each age-group

| AgeGroup | PlayerCount | TotalBetAmount |
| -------- | ----------- | -------------- |
| 45+      | 5,893       | 36,789,802.45  |
| 25-34    | 1,627       | 10,173,954.64  |
| 35-44    | 1,596       | 9,994,364.48   |
| 18-24    | 884         | 5,564,212.62   |

**Insights:**

-   About 60% of all players are in 45+ age-group
-   Players in 45+ age-group has the hghest total bet amount

### Game Popularity and Revenue Analysis

**Objective:** Identify the most popular games and their contribution to revenue.

**Query:**

```sql
	SELECT
		g.GameName,
		gt.TypeName,
		COUNT(b.BetID) AS BetCount,
		SUM(b.BetAmount - b.WinAmount) AS GGR
	FROM Bets b
	JOIN Games g ON b.GameID = g.GameID
	JOIN GameType gt ON g.GameTypeID = gt.GameTypeID
	GROUP BY
		g.GameName,
		gt.TypeName
	ORDER BY
		BetCount DESC,
		GGR DESC;
```

**Result:**

-   The table bellow shows the top five popular games

| GameNam           | TypeName    | BetCount |
| ----------------- | ----------- | -------- |
| Starburst         | Slots       | 4,909    |
| European Roulette | Table Games | 4,892    |
| Gonzo's Quest     | Slots       | 4,869    |
| Bonanza           | Slots       | 4,859    |
| Immortal Romance  | Slots       | 4,820    |

-   The table bellow shows the top games with highest GGR

| GameNam        | TypeName       | GGR        |
| -------------- | -------------- | ---------- |
| Dream Catcher  | Live Casino    | 314,509.06 |
| Super Sic Bo   | Live Casino    | 314,212.40 |
| Caribbean Stud | Poker          | 313,820.02 |
| Live Blackjack | Live Casino    | 312,194.66 |
| Virtual Tennis | Sports Betting | 310,038.20 |

**Insights:**

-   Starburst is the most popular game with highest count of bets
-   Dream Catcher is the game with highest Revenue

### Promotion Effectiveness Analysis

**Objective:** Evaluate the impact of promotions on player activity.

**Query:**

```sql
	SELECT
		pr.PromotionName,
		COUNT(r.RedemptionID) AS RedemptionCount,
		SUM(b.BetAmount) AS TotalBetAmount,
		SUM(b.WinAmount) AS TotalWinAmount
	FROM Promotions pr
	JOIN PromotionRedemption r ON pr.PromotionID = r.PromotionID
	LEFT JOIN Bets b ON r.PlayerID = b.PlayerID AND b.BetDate BETWEEN pr.StartDate AND pr.EndDate
	GROUP BY
		pr.PromotionName
	ORDER BY
		RedemptionCount DESC;
```

**Result:**

-   The table bellow shows promotions, redemption counts and total bets and wins during promotion periods.

| PromotionName                     | RedemptionCount | TotalBetAmount | TotalWinAmount |
| --------------------------------- | --------------- | -------------- | -------------- |
| Lucky Charm Giveaway Special      | 383             | 192149.13      | 176591.18      |
| New Year Bash Bonus Event Special | 352             | 171445.81      | 177063.21      |
| Hunger Games Challenge Special    | 283             | 133078.86      | 129022.72      |
| Mega Win Mania Event              | 242             | 108020.83      | 113146.26      |
| Summer Sizzler Surprise Event     | 229             | 115846.99      | 111031.40      |

**Insights:**

-   The "Lucky Charm Giveaway Special" promotion has the highest redemption counts

### Session Duration and Engagement Trends

**Objective:** Analyze session durations and trends to understand player engagement.

**Query:**

```sql
	SELECT
		s.SessionID,
		p.Name AS PlayerName,
		g.GameName,
		DATEDIFF(MINUTE, s.StartTime, s.EndTime) AS SessionDuration,
		st.TypeName AS SessionType
	FROM Sessions s
	JOIN Players p ON s.PlayerID = p.PlayerID
	JOIN Games g ON s.GameID = g.GameID
	JOIN SessionType st ON s.SessionTypeID = st.SessionTypeID
	ORDER BY SessionDuration DESC;
```

**Result:**

-   The table below summarizes key session engagement metrics, showing session details such as player name, game name, session duration, and session type. It highlights diverse player interactions with different games and session types.

| Session ID | Player Name    | Game Name          | Session Duration (min) | Session Type        |
| ---------- | -------------- | ------------------ | ---------------------- | ------------------- |
| 69648      | Courtney Lutz  | Hot Spin           | 120                    | Tournament Session  |
| 146528     | Mary Hernandez | Vikings Go Berzerk | 120                    | Live Event Session  |
| 182060     | Lisa Anderson  | Virtual Football   | 120                    | Browsing Session    |
| 144342     | Robert Padilla | Gonzo's Quest      | 120                    | Promotional Session |
| 4935       | Amanda Harris  | Divine Fortune     | 120                    | Browsing Session    |

**Insights:**

-   Players engage equally across all session types.
-   All sessions have a consistent duration of 120 minutes.
-   Players show interest in a variety of games.

### Financial Transactions Summarys

**Objective:** Summarize financial transactions by type and status.

**Query:**

```sql
	SELECT
		tt.TypeName AS TransactionType,
		ts.StatusName AS TransactionStatus,
		COUNT(t.TransactionID) AS TransactionCount,
		SUM(t.Amount) AS TotalAmount
	FROM Transactions t
	JOIN TransactionType tt ON t.TransactionTypeID = tt.TransactionTypeID
	JOIN TransactionStatus ts ON t.TransactionStatusID = ts.TransactionStatusID
	GROUP BY
		tt.TypeName,
		ts.StatusName
	ORDER BY TotalAmount DESC;

```

**Result:**

-   The table provides a summary of financial transactions categorized by transaction type, status, count, and total amount. It highlights the distribution of transaction outcomes and their monetary impact within the iGaming platform.

| TransactionType | TransactionStatus | TransactionCount | TotalAmount   |
| --------------- | ----------------- | ---------------- | ------------- |
| Winnings Payout | Cancelled         | 4062             | 20,612,305.32 |
| Winnings Payout | Failed            | 4094             | 20,609,595.76 |
| Deposit         | Failed            | 4047             | 20,371,258.66 |
| Bonus Credit    | Completed         | 4054             | 20,363,755.92 |
| Bonus Credit    | Failed            | 4029             | 20,316,377.15 |

**Insights:**

-   High Volume of Cancelled and Failed Transactions: "Winnings Payout" and "Deposit" transactions have significant instances of cancellations and failures, indicating potential technical or user-related issues.
-   Completed Transactions: "Bonus Credit" and "Withdrawal" transactions contribute significantly to completed financial flows.
-   Pending and Reversed States: A substantial portion of transactions is in pending or reversed states, suggesting further investigation is needed for process optimization.

## Phase 3: Power BI Dashboards

### Player Overview Dashboard

**Description:** Displays player demographics and segmentation.

**Screenshot:**

### Game Performance Dashboard

**Description:** Visualizes game popularity and revenue trends.

**Screenshot:**

### Engagement Trends Dashboard

**Description:** Shows session durations and platform usage over time.

**Screenshot:**

## Key Insights and Recommendations

-   Placeholder for insights and actionable recommendations.

## Conclusion

The iGaming Analytics: Player Behavior, Revenue, and Trends project provides valuable insights into player behavior, game performance, and revenue generation, all of which are crucial for data-driven decision-making in the iGaming industry. The combination of SQL-based analytics and Power BI visualizations presents a comprehensive, actionable view of key performance indicators that can drive strategic decisions for iGaming platforms.
