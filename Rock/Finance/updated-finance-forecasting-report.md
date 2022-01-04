---
tags:
    - language/sql
    - language/lava
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-02 19:18:33
---

# Updated Finance Forecasting Report

> Original idea from Kevin Rutledge - [Recipee Link](https://community.rockrms.com/recipes/27/projected-year-end-giving)  
> Rewritten by Michael Allen to add support for fiscal years, and to fix a few display bugs

This is a fairly complicated report. Read Kevin's recipe (linked above) for a more thorough explanation.

## Dynamic Data Block

### Query

```sql
/* BEGIN Configuration */
DECLARE @Years INT = 4; --How many years to pull? (Including the current year; Changing this will break the lava template)
DECLARE @StartMonth INT = 1; --What month does your FY start on?
DECLARE @AccountList VARCHAR(MAX) = '5,12,24,34,55'; --Comma seperated list of AccountIds to include
/* END Configuration */

DECLARE @StartDate DATE = DATEFROMPARTS( DATEPART( YEAR, GETDATE() ) - ( @Years - 1 ) , @StartMonth, 1 );
DECLARE @EndDate DATE = DATEADD( YEAR, ( @Years - 1 ), @StartDate );

/* If we haven't started this FY yet */
IF ( DATEPART( MONTH, GETDATE() ) < @StartMonth ) BEGIN
	SET @StartDate = DATEADD( YEAR, -1, @StartDate )
	SET @EndDate = DATEADD( YEAR, -1, @EndDate )
END;

DECLARE @Results TABLE( Total DECIMAL(20,2), Year INT, Month INT );

INSERT INTO @Results
SELECT
	SUM( ftd.Amount ) 'Total'
	,DATEPART( YEAR, ft.TransactionDateTime ) 'Year'
	,DATEPART( MONTH, ft.TransactionDateTime ) 'Month'
FROM
	[FinancialTransactionDetail] ftd
	JOIN [FinancialTransaction] ft ON ftd.TransactionId = ft.Id
	--Using an inner join to filter the accounts that are included
	INNER JOIN ufnUtility_CsvToTable( @AccountList ) acct ON ftd.AccountId = acct.Item
WHERE
	ft.TransactionDateTime >= @StartDate
	AND ft.TransactionDateTime < @EndDate
	AND ft.TransactionTypeValueId = 53 --Contribution
GROUP BY
	DATEPART( YEAR, ft.TransactionDateTime )
	,DATEPART( MONTH, ft.TransactionDateTime )
;

/* table1 = current year */
SET @StartDate = @EndDate;
SET @EndDate = DATEFROMPARTS( DATEPART( YEAR, GETDATE() ), DATEPART( MONTH, GETDATE() ), 1 );

SELECT(
	SELECT
		SUM( ftd.Amount ) 'Total'
		,DATEPART( YEAR, ft.TransactionDateTime ) 'Year'
		,DATEPART( MONTH, ft.TransactionDateTime ) 'Month'
	FROM
		[FinancialTransactionDetail] ftd
		JOIN [FinancialTransaction] ft ON ftd.TransactionId = ft.Id
		--Using an inner join to filter the accounts that are included
		INNER JOIN ufnUtility_CsvToTable( @AccountList ) acct ON ftd.AccountId = acct.Item
	WHERE
		ft.TransactionDateTime >= @StartDate
		AND ft.TransactionDateTime < @EndDate
		AND ft.TransactionTypeValueId = 53 --Contribution
	GROUP BY
		DATEPART( YEAR, ft.TransactionDateTime )
		,DATEPART( MONTH, ft.TransactionDateTime )
	ORDER BY 'Year', 'Month'
	FOR JSON PATH
) 'json';

/* table2, table3, table4 = previous 3 years*/
DECLARE @i INT = 0;

WHILE @i < ( @Years - 1 ) BEGIN
	SELECT (
		SELECT * FROM @Results
		ORDER BY 'Year', 'Month'
		OFFSET ( @i * 12 ) ROWS
		FETCH NEXT 12 ROWS ONLY
		FOR JSON PATH
	) 'json';

    SET @i = @i + 1;
END;

/* table5 = totals */
SELECT(
	SELECT
		SUM( Total ) 'Total'
		,Month
		,MIN( Year ) 'Year'
	FROM @Results
	GROUP BY Month
	ORDER BY 'Year'
	FOR JSON PATH
) 'json';
```

### Formatted Output

```liquid
{% assign current   = table1.rows[0].json | FromJSON %}
{% assign 3yearsAgo = table2.rows[0].json | FromJSON %}
{% assign 2yearsAgo = table3.rows[0].json | FromJSON %}
{% assign 1yearsAgo = table4.rows[0].json | FromJSON %}
{% assign totals    = table5.rows[0].json | FromJSON %}

{% assign 3yearsAgoTotal = 0 %}
{% assign 2yearsAgoTotal = 0 %}
{% assign 1yearsAgoTotal = 0 %}
{% assign currentTotal   = 0 %}

{% for row in 3yearsAgo %}
    {% assign 3yearsAgoTotal = 3yearsAgoTotal | Plus:row.Total %}
{% endfor %}
{% for row in 2yearsAgo %}
    {% assign 2yearsAgoTotal = 2yearsAgoTotal | Plus:row.Total %}
{% endfor %}
{% for row in 1yearsAgo %}
    {% assign 1yearsAgoTotal = 1yearsAgoTotal | Plus:row.Total %}
{% endfor %}
{% for row in current %}
    {% assign currentTotal = currentTotal | Plus:row.Total %}
{% endfor %}
{% assign grandTotal = 3yearsAgoTotal | Plus:2yearsAgoTotal | Plus:1yearsAgoTotal %}

{% capture multipliers %}
    [
    {% for row in totals %}
        {{ row.Total | DividedBy:grandTotal,4 }}
        {% unless forloop.last %},{% endunless %}
    {% endfor %}
    ]
{% endcapture %}
{% assign multipliers = multipliers | FromJSON %}

<style>
.year {
    text-align:center;
    font-weight: bold;
}
.text-right {
    font-family: monospace;
    text-align: right;
}
.table {
    font-size: 0.9em;
}
</style>

<div class="row">
    <div class="col-lg-3 col-md-6">
        <h5 class="year">{{ 3yearsAgo | Select:'Year' | Uniq | Join:'-' }}</h5>
        <table class="text-right table">
            <tr>
                <th class="text-right">Month</th>
                <th class="text-right">Total Given</th>
                <th class="text-right">% of Total</th>
            </tr>
{% for month in 3yearsAgo %}
            <tr>
                <td>{{ month.Month | Append:'/01/' | Append:month.Year | Date:'MMM. yy' }}</td>
                <td>{{ month.Total | FormatAsCurrency }}</td>
                <td>{{ month.Total | DividedBy:3yearsAgoTotal,4 | Format:'p' }}</td>
            </tr>
{% endfor %}
        </table>
    </div>
    <div class="col-lg-3 col-md-6">
        <h5 class="year">{{ 2yearsAgo | Select:'Year' | Uniq | Join:'-' }}</h5>
        <table class="text-right table">
            <tr>
                <th class="text-right">Month</th>
                <th class="text-right">Total Given</th>
                <th class="text-right">% of Total</th>
            </tr>
{% for month in 2yearsAgo %}
            <tr>
                <td>{{ month.Month | Append:'/01/' | Append:month.Year | Date:'MMM. yy' }}</td>
                <td>{{ month.Total | FormatAsCurrency }}</td>
                <td>{{ month.Total | DividedBy:2yearsAgoTotal,4 | Format:'p' }}</td>
            </tr>
{% endfor %}
        </table>
    </div>
    <div class="col-lg-3 col-md-6">
        <h5 class="year">{{ 1yearsAgo | Select:'Year' | Uniq | Join:'-' }}</h5>
        <table class="text-right table">
            <tr>
                <th class="text-right">Month</th>
                <th class="text-right">Total Given</th>
                <th class="text-right">% of Total</th>
            </tr>
{% for month in 1yearsAgo %}
            <tr>
                <td>{{ month.Month | Append:'/01/' | Append:month.Year | Date:'MMM. yy' }}</td>
                <td>{{ month.Total | FormatAsCurrency }}</td>
                <td>{{ month.Total | DividedBy:1yearsAgoTotal,4 | Format:'p' }}</td>
            </tr>
{% endfor %}
        </table>
    </div>
    <div class="col-lg-3 col-md-6">
        <h5 class="year">3 Year Total</h5>
        <table class="text-right table">
            <tr>
                <th class="text-right">Month</th>
                <th class="text-right">Total Given</th>
                <th class="text-right">% of Total</th>
            </tr>
{% for month in totals %}
            <tr>
                <td>{{ month.Month | Append:'/01/2000' | Date:'MMM.' }}</td>
                <td>{{ month.Total | FormatAsCurrency }}</td>
                <td>{{ multipliers[forloop.index0] | Format:'p' }}</td>
            </tr>
{% endfor %}
        </table>
    </div>
</div>
<div class="row">
    <div class="col-md-6">
        <h5 class="year">Current Year: {{ current | Select:'Year' | Uniq | Join:'-' }} </h5>
{% if currentTotal == 0 %}
        <p>At least 1 month must be complete.</p>
{% else %}
        <table class="text-right table">
            <tr>
                <th class="text-right">Month</th>
                <th class="text-right">Total Given</th>
                <th class="text-right">% Multiplier</th>
                <th class="text-right">Year End Projection</th>
            </tr>
    {% assign cumulativePercent = 0 %}
    {% for month in current %}
        {% assign cumulativePercent = cumulativePercent | Plus:multipliers[forloop.index0] %}
            <tr>
                <td>{{ month.Month | Append:'/01/' | Append:month.Year | Date:'MMM. yy' }}</td>
                <td>{{ month.Total | FormatAsCurrency }}</td>
                <td>{{ multipliers[forloop.index0] | Format:'p' }}</td>
                <td>{{ month.Total | DividedBy:multipliers[forloop.index0] | FormatAsCurrency }}</td>
            </tr>
    {% endfor %}
        </table>
{% endif %}
    </div>
    <div class="col-lg-3 col-md-6">
        <h5 class="year">Yearly Totals</h5>
        <table class="text-right table">
            <tr>
                <th class="text-right">Year</th>
                <th class="text-right">Total</th>
            </tr>
            <tr>
                <td>{{ 3yearsAgo | Select:'Year' | Uniq | Join:'-' }}</td>
                <td>{{ 3yearsAgoTotal | FormatAsCurrency }}</td>
            </tr>
            <tr>
                <td>{{ 2yearsAgo | Select:'Year' | Uniq | Join:'-' }}</td>
                <td>{{ 2yearsAgoTotal | FormatAsCurrency }}</td>
            </tr>
            <tr>
                <td>{{ 1yearsAgo | Select:'Year' | Uniq | Join:'-' }}</td>
                <td>{{ 1yearsAgoTotal | FormatAsCurrency }}</td>
            </tr>
{% if currentTotal > 0 %}
            <tr>
                <td>{{ current | Select:'Year' | Uniq | Join:'-' }}</td>
                <td>{{ currentTotal | FormatAsCurrency }}</td>
            </tr>
{% endif %}
        </table>
        <div class="alert-success alert"  style="margin-top:20px;">
            <h5>{{ current | Select:'Year' | Uniq | Join:'-' }} Year End Projection:</h5>
{% if currentTotal == 0 %}
            <p>At least 1 month must be complete.</p>
{% else %}
            <p>{{ currentTotal | DividedBy:cumulativePercent | FormatAsCurrency }}</p>
{% endif %}
        </div>
    </div>
</div>
<div class="row">
{% assign currentSize = current | Size | Minus:1 %}
{% capture chartData %}
    [
    {% for i in (0..11) %}
        {
            name: '{{ 3yearsAgo[i].Month | Append:'/01/' | Append:3yearsAgo[i].Year | Date:'MMM.' }}'
            ,3year: '{{ 3yearsAgo[i].Total }}'
            ,2year: '{{ 2yearsAgo[i].Total }}'
            ,1year: '{{ 1yearsAgo[i].Total }}'
        {% unless i > currentSize %}
            ,current: '{{ current[i].Total }}'
        {% endunless %}
        }
        {% unless forloop.last %},{% endunless %}
    {% endfor %}
    ]
{% endcapture %}
{% assign chartData = chartData | FromJSON %}

    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.7.1/Chart.bundle.min.js"> </script>
    <div class="chart-container" style="position:relative;height:300px;width:100%;margin-bottom:50px;">
        <h5>Giving History By Month</h5>
        <canvas id="myChart"></canvas>
    </div>
</div>

<script>
    var ctx = document.getElementById("myChart").getContext('2d');
    var myChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: [ "{{ chartData | Select:'name' | Join:'","' }}" ],
            datasets: [
                {
                    label: '{{ 3yearsAgo | Select:'Year' | Uniq | Join:'-' }}',
                    fill: false,
                    data:  [ {{ chartData | Select:'3year' | Join:',' }} ],
                    backgroundColor: [ "rgba(54,162,235,1)" ],
                    borderColor: [ "rgba(54,162,235,1)" ],
                    borderWidth: 1
                },
                {
                    label: '{{ 2yearsAgo | Select:'Year' | Uniq | Join:'-' }}',
                    fill: false,
                    data:  [ {{ chartData | Select:'2year' | Join:',' }} ],
                    backgroundColor: [ "rgba(75,192,192,1)" ],
                    borderColor: [ "rgba(75,192,192,1)" ],
                    borderWidth: 1
                },
                {
                    label: '{{ 1yearsAgo | Select:'Year' | Uniq | Join:'-' }}',
                    fill: false,
                    data:  [ {{ chartData | Select:'1year' | Join:',' }} ],
                    backgroundColor: [ "rgba(255,159,64,1)" ],
                    borderColor: [ "rgba(255,159,64,1)" ],
                    borderWidth: 1
                },
                {
                    label: '{{ current | Select:'Year' | Uniq | Join:'-' }}',
                    fill: false,
                    data: [ {{ chartData | Select:'current' | Join:',' }} ],
                    backgroundColor: [ "rgba(153,102,255,1)" ],
                    borderColor: [ "rgba(153,102,255,1)" ],
                    borderWidth: 1
                },
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            animation: {
                duration: 2500,
            },
            legend: {
                position: 'right'
            },
            scales: {
                yAxes: [{
                    ticks: {
                        beginAtZero:false,
                        callback: function(value, index, values) {
                            return '$' + value/1000 + 'k';
                        }
                    }
                }]
            },
            tooltips: {
                callbacks: {
                    label: function(tooltipItem, data) {
                        var label = data.datasets[tooltipItem.datasetIndex].label || '';

                        if (label) {
                            label += ': ';
                        }
                        label += '$' + Math.round(tooltipItem.yLabel / 100) / 10 + 'k';
                        return label;
                    }
                }
            }
        }
    });
</script>
```
