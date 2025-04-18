---
title: Advanced Economic Metrics
theme: dashboard
toc: true
---

```js
const INDICATORS = FileAttachment("data/indicators.json").json();
console.log("Loaded INDICATORS:", INDICATORS);
```

```js
// Define advanced composite economic indicators
const ADVANCED_INDICATORS = {
  MISERY_INDEX: {
    id: "MISERY_INDEX",
    title: "Misery Index",
    description:
      "The Misery Index is like a 'how hard is life right now?' score. It adds together two things: how many people can't find jobs (unemployment) and how much more expensive things are getting (inflation). The higher the number, the harder it is for regular people to make ends meet.",
    formula: "Unemployment Rate + Inflation Rate",
    components: ["UNRATE", "CPIAUCSL"],
    units: "Percentage Points",
    group: "Composite Indicators"
  },
  HOUSING_AFFORDABILITY: {
    id: "HOUSING_AFFORDABILITY",
    title: "Housing Affordability Index",
    description:
      "This index shows how affordable housing is for the average person. It compares median household income to median home prices. A higher number means housing is more affordable, while a lower number means it's harder for people to buy homes.",
    formula: "Median Household Income / Median Home Price × 100",
    components: ["MEHOINUSA672N", "MSPUS"],
    units: "Index (100 = baseline affordability)",
    group: "Housing Indicators"
  },
  ALABAMA_HOUSING_AFFORDABILITY: {
    id: "ALABAMA_HOUSING_AFFORDABILITY",
    title: "Alabama Housing Affordability Index",
    description:
      "This index shows how affordable housing is for the average person in Alabama. It compares median household income to median home prices in Alabama. A higher number means housing is more affordable, while a lower number means it's harder for people to buy homes in Alabama.",
    formula: "Alabama Median Household Income / Alabama Median Home Price × 100",
    components: ["MEHOINUSALA646N", "MEDLISPRIAL"],
    units: "Index (100 = baseline affordability)",
    group: "Housing Indicators"
  },
  REAL_INTEREST_RATE: {
    id: "REAL_INTEREST_RATE",
    title: "Real Interest Rate",
    description:
      "The Real Interest Rate shows how much money you're really making (or losing) when you save money in the bank. It's the interest rate minus inflation. A positive number means your savings are growing in real terms, while a negative number means inflation is eating away at your savings.",
    formula: "Nominal Interest Rate - Inflation Rate",
    components: ["FEDFUNDS", "CPIAUCSL"],
    units: "Percentage Points",
    group: "Financial Indicators"
  },
  ECONOMIC_GROWTH_INDEX: {
    id: "ECONOMIC_GROWTH_INDEX",
    title: "Economic Growth Index",
    description:
      "This index combines GDP growth with job creation to give a simple picture of how the economy is growing. It helps show if the economy is getting stronger or weaker over time.",
    formula: "GDP Growth Rate + Employment Growth Rate",
    components: ["GDPC1", "UNRATE"],
    units: "Percentage Points",
    group: "Growth Indicators"
  },
  HOUSING_DIVERGENCE: {
    id: "HOUSING_DIVERGENCE",
    title: "Housing Affordability Divergence",
    description:
      "Shows how Alabama's housing affordability differs from the national trend. Positive values indicate Alabama is more affordable than the national average, negative values indicate it's less affordable.",
    formula: "Alabama Affordability Index - National Affordability Index",
    components: ["HOUSING_AFFORDABILITY", "ALABAMA_HOUSING_AFFORDABILITY"],
    units: "Percentage Points",
    group: "Housing Indicators"
  }
};

// Load the indicators data
const INDICATORS_FLAT = [];

// Set default start date to 5 years ago
const DEFAULT_START_DATE = new Date();
DEFAULT_START_DATE.setFullYear(DEFAULT_START_DATE.getFullYear() - 9);

// Process each indicator
for (const [key, value] of Object.entries(INDICATORS)) {
  console.log(`Processing indicator: ${key}`, value);
  if (Array.isArray(value)) {
    // Handle array of indicators
    value.forEach((data) => {
      INDICATORS_FLAT.push({
        Indicator: data.id,
        Value: Number(data.value),
        Date: new Date(data.date),
        Units: data.units,
        [data.units]: data.value,
        Title: data.title,
        Description: data.description,
        Group: data.group
      });
    });
  } else if (typeof value === "object" && value !== null) {
    // Handle single indicator
    INDICATORS_FLAT.push({
      Indicator: value.id,
      Value: Number(value.value),
      Date: new Date(value.date),
      Units: value.units,
      [value.units]: value.value,
      Title: value.title,
      Description: value.description,
      Group: value.group
    });
  }
}

// Sort by date
INDICATORS_FLAT.sort((a, b) => a.Date.getTime() - b.Date.getTime());
console.log("Flattened INDICATORS_FLAT:", INDICATORS_FLAT);

// Function to filter data by date range
function filterDataByDate(data, startDate) {
  if (!startDate) return data;
  return data.filter((d) => d.Date >= startDate);
}

// Function to calculate the Misery Index
function calculateMiseryIndex(unemploymentData, inflationData, startDate = DEFAULT_START_DATE) {
  // Filter data by start date
  unemploymentData = filterDataByDate(unemploymentData, startDate);
  inflationData = filterDataByDate(inflationData, startDate);

  // Ensure both datasets have the same dates
  const commonDates = unemploymentData
    .map((d) => d.Date)
    .filter((date) => inflationData.some((d) => d.Date.getTime() === date.getTime()));

  return commonDates.map((date) => {
    const unemployment = unemploymentData.find((d) => d.Date.getTime() === date.getTime())?.Value || 0;
    const inflation = inflationData.find((d) => d.Date.getTime() === date.getTime())?.Value || 0;

    return {
      Date: date,
      Value: unemployment + inflation,
      Unemployment: unemployment,
      Inflation: inflation
    };
  });
}

// Function to calculate Housing Affordability
function calculateHousingAffordability(incomeData, priceData, startDate = DEFAULT_START_DATE) {
  // Filter data by start date
  incomeData = filterDataByDate(incomeData, startDate);
  priceData = filterDataByDate(priceData, startDate);

  console.log("Calculating Housing Affordability with:", {
    incomeData,
    priceData
  });

  // Ensure both datasets have the same dates
  const commonDates = incomeData
    .map((d) => d.Date)
    .filter((date) => priceData.some((d) => d.Date.getTime() === date.getTime()));

  console.log("Common dates found:", commonDates);

  const result = commonDates.map((date) => {
    const income = incomeData.find((d) => d.Date.getTime() === date.getTime())?.Value || 0;
    const price = priceData.find((d) => d.Date.getTime() === date.getTime())?.Value || 0;

    // Calculate affordability index (higher is more affordable)
    const affordability = price > 0 ? (income / price) * 100 : 0;

    return {
      Date: date,
      Value: affordability,
      Income: income,
      Price: price
    };
  });

  console.log("Calculated affordability result:", result);
  return result;
}

// Function to calculate Real Interest Rate
function calculateRealInterestRate(interestData, inflationData, startDate = DEFAULT_START_DATE) {
  // Filter data by start date
  interestData = filterDataByDate(interestData, startDate);
  inflationData = filterDataByDate(inflationData, startDate);

  // Ensure both datasets have the same dates
  const commonDates = interestData
    .map((d) => d.Date)
    .filter((date) => inflationData.some((d) => d.Date.getTime() === date.getTime()));

  return commonDates.map((date) => {
    const interest = interestData.find((d) => d.Date.getTime() === date.getTime())?.Value || 0;
    const inflation = inflationData.find((d) => d.Date.getTime() === date.getTime())?.Value || 0;

    return {
      Date: date,
      Value: interest - inflation,
      Interest: interest,
      Inflation: inflation
    };
  });
}

// Function to calculate an exponentially weighted moving average
function calculateEWMA(data, smoothingFactor = 0.2) {
  if (!data || data.length === 0) return [];

  // Sort data by date to ensure chronological order
  const sortedData = [...data].sort((a, b) => a.Date.getTime() - b.Date.getTime());

  // Initialize the result array with the first data point
  const result = [
    {
      Date: sortedData[0].Date,
      Value: sortedData[0].Value
    }
  ];

  // Calculate EWMA for each subsequent point
  for (let i = 1; i < sortedData.length; i++) {
    const currentValue = sortedData[i].Value;
    const previousEWMA = result[i - 1].Value;

    // EWMA formula: EMA = α × current + (1 - α) × previous EMA
    const ewma = smoothingFactor * currentValue + (1 - smoothingFactor) * previousEWMA;

    result.push({
      Date: sortedData[i].Date,
      Value: ewma
    });
  }

  return result;
}

// Function to render an advanced indicator
function renderAdvancedIndicator(indicator) {
  if (indicator.id === "MISERY_INDEX") {
    // Get the component data
    const unemploymentData = INDICATORS_FLAT.filter((item) => item.Indicator === "UNRATE");
    const inflationData = INDICATORS_FLAT.filter((item) => item.Indicator === "CPIAUCSL");

    // Calculate the Misery Index with date filter
    const miseryIndexData = calculateMiseryIndex(unemploymentData, inflationData);

    // Calculate the exponentially weighted moving average
    const ewmaData = calculateEWMA(miseryIndexData, 0.5);

    // Create the plot with both a line and an EWMA line
    return Plot.plot({
      title: indicator.title,
      subtitle: indicator.description,
      x: { grid: true, label: "Date" },
      y: { grid: true, label: indicator.units },
      width: 800,
      height: 400,
      margin: 40,
      marks: [
        // Original data line
        Plot.lineY(miseryIndexData, {
          x: "Date",
          y: "Value",
          stroke: "red",
          strokeWidth: 1.5
        }),
        // EWMA line
        Plot.lineY(ewmaData, {
          x: "Date",
          y: "Value",
          stroke: "red",
          strokeWidth: 2,
          strokeDasharray: "5,5"
        })
      ]
    });
  } else if (indicator.id === "HOUSING_AFFORDABILITY") {
    // Get the component data for both national and Alabama
    const nationalIncomeData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEHOINUSA672N");
    const nationalPriceData = INDICATORS_FLAT.filter((item) => item.Indicator === "MSPUS");
    const alabamaIncomeData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEHOINUSALA646N");
    const alabamaPriceData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEDLISPRIAL");

    // Calculate Housing Affordability for both with date filter
    const nationalAffordabilityData = calculateHousingAffordability(nationalIncomeData, nationalPriceData);
    const alabamaAffordabilityData = calculateHousingAffordability(alabamaIncomeData, alabamaPriceData);

    // Calculate the exponentially weighted moving average for both
    const nationalEwmaData = calculateEWMA(nationalAffordabilityData, 0.65);
    const alabamaEwmaData = calculateEWMA(alabamaAffordabilityData, 0.65);

    // Get date range for subtitle
    const allDates = [...nationalAffordabilityData, ...alabamaAffordabilityData].map((d) => d.Date);
    const startDate = new Date(Math.min(...allDates.map((d) => d.getTime())));
    const endDate = new Date(Math.max(...allDates.map((d) => d.getTime())));
    const dateRange = `Data from ${startDate.toLocaleDateString()} to ${endDate.toLocaleDateString()}`;

    // Create the plot with both national and Alabama data
    return Plot.plot({
      title: "Housing Affordability Comparison",
      subtitle: `Comparing national and Alabama housing affordability indices. Higher values indicate more affordable housing.\n${dateRange}`,
      x: {
        grid: true,
        label: "Date",
        tickRotate: -45,
        tickFormat: (d) => d.toLocaleDateString()
      },
      y: {
        grid: true,
        label: indicator.units,
        nice: true,
        domain: [15, 35],
        tickFormat: (d) => d.toFixed(1)
      },
      width: 800,
      height: 400,
      margin: 60,
      marks: [
        // National data line
        Plot.lineY(nationalAffordabilityData, {
          x: "Date",
          y: "Value",
          stroke: "blue",
          strokeWidth: 1.5,
          title: (d) => `National: ${d.Value.toFixed(1)}`
        }),
        // National EWMA line
        Plot.lineY(nationalEwmaData, {
          x: "Date",
          y: "Value",
          stroke: "blue",
          strokeWidth: 2,
          strokeDasharray: "5,5"
        }),
        // National data points
        Plot.dot(nationalAffordabilityData, {
          x: "Date",
          y: "Value",
          fill: "blue",
          r: 2,
          title: (d) => `National: ${d.Value.toFixed(1)}\nDate: ${d.Date.toLocaleDateString()}`
        }),
        // Alabama data line
        Plot.lineY(alabamaAffordabilityData, {
          x: "Date",
          y: "Value",
          stroke: "purple",
          strokeWidth: 1.5,
          title: (d) => `Alabama: ${d.Value.toFixed(1)}`
        }),
        // Alabama EWMA line
        Plot.lineY(alabamaEwmaData, {
          x: "Date",
          y: "Value",
          stroke: "purple",
          strokeWidth: 2,
          strokeDasharray: "5,5"
        }),
        // Alabama data points
        Plot.dot(alabamaAffordabilityData, {
          x: "Date",
          y: "Value",
          fill: "purple",
          r: 2,
          title: (d) => `Alabama: ${d.Value.toFixed(1)}\nDate: ${d.Date.toLocaleDateString()}`
        }),
        // Legend
        Plot.text(
          [
            { x: startDate, y: 33, text: "National", fill: "blue" },
            { x: startDate, y: 31, text: "Alabama", fill: "purple" }
          ],
          {
            x: "x",
            y: "y",
            text: "text",
            fill: "fill",
            dx: 50
          }
        )
      ]
    });
  } else if (indicator.id === "ALABAMA_HOUSING_AFFORDABILITY") {
    // Get the component data
    const incomeData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEHOINUSALA646N");
    const priceData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEDLISPRIAL");

    // Calculate Housing Affordability with date filter
    const affordabilityData = calculateHousingAffordability(incomeData, priceData);

    // Calculate the exponentially weighted moving average
    const ewmaData = calculateEWMA(affordabilityData, 0.6);

    // Get date range for subtitle
    const dateRange =
      affordabilityData.length > 0
        ? `Data from ${affordabilityData[0].Date.toLocaleDateString()} to ${affordabilityData[affordabilityData.length - 1].Date.toLocaleDateString()}`
        : "No data available";

    // Create the plot with both a line and an EWMA line
    return Plot.plot({
      title: indicator.title,
      subtitle: `${indicator.description}\n${dateRange}`,
      x: {
        grid: true,
        label: "Date",
        tickRotate: -45,
        tickFormat: (d) => d.toLocaleDateString()
      },
      y: {
        grid: true,
        label: indicator.units,
        nice: true
      },
      width: 800,
      height: 400,
      margin: 60,
      marks: [
        // Original data line
        Plot.lineY(affordabilityData, {
          x: "Date",
          y: "Value",
          stroke: "purple",
          strokeWidth: 1.5
        }),
        // EWMA line
        Plot.lineY(ewmaData, {
          x: "Date",
          y: "Value",
          stroke: "purple",
          strokeWidth: 2,
          strokeDasharray: "5,5"
        }),
        // Data points
        Plot.dot(affordabilityData, {
          x: "Date",
          y: "Value",
          fill: "purple",
          r: 3
        })
      ]
    });
  } else if (indicator.id === "REAL_INTEREST_RATE") {
    // Get the component data
    const interestData = INDICATORS_FLAT.filter((item) => item.Indicator === "FEDFUNDS");
    const inflationData = INDICATORS_FLAT.filter((item) => item.Indicator === "CPIAUCSL");

    // Calculate Real Interest Rate with date filter
    const realInterestData = calculateRealInterestRate(interestData, inflationData);

    // Calculate the exponentially weighted moving average
    const ewmaData = calculateEWMA(realInterestData, 0.2);

    // Create the plot with both a line and an EWMA line
    return Plot.plot({
      title: indicator.title,
      subtitle: indicator.description,
      x: { grid: true, label: "Date" },
      y: { grid: true, label: indicator.units },
      width: 800,
      height: 400,
      margin: 40,
      marks: [
        // Original data line
        Plot.lineY(realInterestData, {
          x: "Date",
          y: "Value",
          stroke: "green",
          strokeWidth: 1.5
        }),
        // EWMA line
        Plot.lineY(ewmaData, {
          x: "Date",
          y: "Value",
          stroke: "green",
          strokeWidth: 2,
          strokeDasharray: "5,5"
        })
      ]
    });
  } else if (indicator.id === "HOUSING_DIVERGENCE") {
    // Get the component data for both national and Alabama
    const nationalIncomeData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEHOINUSA672N");
    const nationalPriceData = INDICATORS_FLAT.filter((item) => item.Indicator === "MSPUS");
    const alabamaIncomeData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEHOINUSALA646N");
    const alabamaPriceData = INDICATORS_FLAT.filter((item) => item.Indicator === "MEDLISPRIAL");

    // Calculate Housing Affordability for both with date filter
    const nationalAffordabilityData = calculateHousingAffordability(nationalIncomeData, nationalPriceData);
    const alabamaAffordabilityData = calculateHousingAffordability(alabamaIncomeData, alabamaPriceData);

    // Calculate divergence
    const commonDates = nationalAffordabilityData
      .map((d) => d.Date.getTime())
      .filter((date) => alabamaAffordabilityData.some((d) => d.Date.getTime() === date));

    const divergenceData = commonDates.map((date) => {
      const national = nationalAffordabilityData.find((d) => d.Date.getTime() === date);
      const alabama = alabamaAffordabilityData.find((d) => d.Date.getTime() === date);
      return {
        Date: new Date(date),
        Value: alabama.Value - national.Value,
        NationalValue: national.Value,
        AlabamaValue: alabama.Value
      };
    });

    // Calculate moving average for smoother trend
    const ewmaDivergence = calculateEWMA(divergenceData, 0.65);

    // Get date range for subtitle
    const startDate = new Date(Math.min(...divergenceData.map((d) => d.Date.getTime())));
    const endDate = new Date(Math.max(...divergenceData.map((d) => d.Date.getTime())));
    const dateRange = `Data from ${startDate.toLocaleDateString()} to ${endDate.toLocaleDateString()}`;

    // Create the divergence plot
    return Plot.plot({
      title: "Housing Affordability Divergence: Alabama vs National",
      subtitle: `Shows how Alabama's housing affordability differs from the national average.\nPositive values mean Alabama is more affordable. ${dateRange}`,
      x: {
        grid: true,
        label: "Date",
        tickRotate: -45,
        tickFormat: (d) => d.toLocaleDateString()
      },
      y: {
        grid: true,
        label: "Difference in Affordability Index",
        domain: [-5, 5],
        tickFormat: (d) => d.toFixed(1)
      },
      width: 800,
      height: 400,
      margin: 60,
      marks: [
        // Zero line
        Plot.ruleY([0], {
          stroke: "#ccc",
          strokeWidth: 1
        }),
        // Divergence area
        Plot.areaY(divergenceData, {
          x: "Date",
          y: "Value",
          fill: (d) => (d.Value >= 0 ? "rgba(0, 128, 0, 0.1)" : "rgba(255, 0, 0, 0.1)")
        }),
        // Divergence line
        Plot.lineY(divergenceData, {
          x: "Date",
          y: "Value",
          stroke: (d) => (d.Value >= 0 ? "green" : "red"),
          strokeWidth: 1.5,
          title: (d) =>
            `Difference: ${d.Value.toFixed(1)}\nAlabama: ${d.AlabamaValue.toFixed(1)}\nNational: ${d.NationalValue.toFixed(1)}`
        }),
        // Moving average line
        Plot.lineY(ewmaDivergence, {
          x: "Date",
          y: "Value",
          stroke: "black",
          strokeWidth: 2,
          strokeDasharray: "5,5"
        }),
        // Data points
        Plot.dot(divergenceData, {
          x: "Date",
          y: "Value",
          fill: (d) => (d.Value >= 0 ? "green" : "red"),
          r: 2,
          title: (d) => `Difference: ${d.Value.toFixed(1)}\nDate: ${d.Date.toLocaleDateString()}`
        })
      ]
    });
  }

  // Add more advanced indicators here as needed
  return null;
}
```

## Advanced Economic Indicators

### Misery Index

<div class="card">
  <div class="card-chart">
    ${renderAdvancedIndicator(ADVANCED_INDICATORS.MISERY_INDEX)}
  </div>
</div>

### Housing Affordability Comparison

<div class="card">
  <div class="card-chart">
    ${renderAdvancedIndicator(ADVANCED_INDICATORS.HOUSING_AFFORDABILITY)}
  </div>
</div>

### Housing Affordability Divergence

<div class="card">
  <div class="card-chart">
    ${renderAdvancedIndicator(ADVANCED_INDICATORS.HOUSING_DIVERGENCE)}
  </div>
</div>

### Real Interest Rate

<div class="card">
  <div class="card-chart">
    ${renderAdvancedIndicator(ADVANCED_INDICATORS.REAL_INTEREST_RATE)}
  </div>
</div>

<style>
.grid {
  display: grid;
  gap: 1.5rem;
  margin: 2rem 0;
}

.grid-cols-2 {
  grid-template-columns: repeat(2, 1fr);
}

@media (max-width: 1024px) {
  .grid-cols-2 {
    grid-template-columns: 1fr;
  }
}

.card {
  background: var(--theme-background-secondary);
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05),
              0 1px 3px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(0, 0, 0, 0.08),
              0 2px 4px rgba(0, 0, 0, 0.12);
}

.card-header h3 {
  margin: 0 0 1.5rem 0;
  color: var(--theme-foreground-focus);
  font-size: 1.4rem;
  font-weight: 600;
  letter-spacing: -0.02em;
}

.card-header p {
  margin: 0 0 1.5rem 0;
  color: var(--theme-foreground-muted);
  font-size: 1rem;
  line-height: 1.5;
}

.card-chart {
  margin: 1rem 0;
  padding: 1rem;
  background: var(--theme-background-primary);
  border-radius: 8px;
  overflow: hidden;
}

.dashboard-grid {
  display: flex;
  flex-direction: column;
  gap: 3rem;
  margin: 2rem 0;
  padding: 0 1rem;
}

section {
  margin-bottom: 4rem;
}

section h2 {
  margin-bottom: 2rem;
  color: var(--theme-foreground-focus);
  font-size: 2rem;
  font-weight: 700;
  letter-spacing: -0.03em;
  position: relative;
  padding-bottom: 0.5rem;
}

section h2::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 60px;
  height: 4px;
  background: var(--theme-primary);
  border-radius: 2px;
}

/* Add a subtle gradient background to the page */
body {
  background: linear-gradient(
    to bottom,
    var(--theme-background-primary),
    var(--theme-background-secondary)
  );
  min-height: 100vh;
}

/* Make the plots more visually appealing */
svg {
  border-radius: 8px;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .card {
    padding: 1.5rem;
  }
  
  .card-header h3 {
    font-size: 1.2rem;
  }
  
  section h2 {
    font-size: 1.6rem;
  }
}
</style>

```

```
