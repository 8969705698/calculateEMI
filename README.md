# calculateEMI
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>EMI Calculator</title>
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
  <script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>

<div id="app" class="container mt-5">
  <div class="row">
    <div class="col-md-6">
      <h2>Loan Details</h2>
      <form @submit.prevent="calculateEMI">
        <div class="form-group">
          <label for="loanAmount">Loan Amount:</label>
          <input type="number" v-model="loanAmount" class="form-control" required>
        </div>
        <div class="form-group">
          <label for="interestRate">Interest Rate (%):</label>
          <input type="number" v-model="interestRate" class="form-control" required>
        </div>
        <div class="form-group">
          <label for="loanTenure">Loan Tenure (months):</label>
          <input type="number" v-model="loanTenure" class="form-control" required>
        </div>
        <button type="submit" class="btn btn-primary">Calculate EMI</button>
      </form>
    </div>
    <div class="col-md-6">
      <h2>EMI Details</h2>
      <p>EMI: {{ emi.toFixed(2) }}</p>
      <p>Interest Payable: {{ interestPayable.toFixed(2) }}</p>
      <p>Total Payable: {{ totalPayable.toFixed(2) }}</p>
      <canvas id="pieChart" width="400" height="400"></canvas>
    </div>
  </div>

  <div class="row mt-5">
    <div class="col-md-12">
      <h2>Amortization Schedule</h2>
      <table class="table">
        <thead>
          <tr>
            <th>Payment #</th>
            <th>EMI</th>
            <th>Principal</th>
            <th>Interest</th>
            <th>Balance</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="payment in amortizationSchedule" :key="payment.paymentNumber">
            <td>{{ payment.paymentNumber }}</td>
            <td>{{ payment.emi.toFixed(2) }}</td>
            <td>{{ payment.principal.toFixed(2) }}</td>
            <td>{{ payment.interest.toFixed(2) }}</td>
            <td>{{ payment.balance.toFixed(2) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</div>

<script>
new Vue({
  el: '#app',
  data: {
    loanAmount: 0,
    interestRate: 0,
    loanTenure: 0,
    emi: 0,
    interestPayable: 0,
    totalPayable: 0,
    amortizationSchedule: []
  },
  methods: {
    calculateEMI: function() {
      const principal = this.loanAmount;
      const rateOfInterest = this.interestRate / 1200; // Monthly interest rate
      const numberOfPayments = this.loanTenure;

      // Calculate EMI using the formula
      this.emi = (principal * rateOfInterest) / (1 - Math.pow(1 + rateOfInterest, -numberOfPayments));
      this.interestPayable = this.emi * numberOfPayments - principal;
      this.totalPayable = principal + this.interestPayable;

      // Generate amortization schedule
      this.amortizationSchedule = [];
      let balance = principal;
      for (let i = 1; i <= numberOfPayments; i++) {
        const interest = balance * rateOfInterest;
        const principalPayment = this.emi - interest;
        balance -= principalPayment;
        this.amortizationSchedule.push({
          paymentNumber: i,
          emi: this.emi,
          principal: principalPayment,
          interest: interest,
          balance: balance
        });
      }

      // Update Pie Chart
      this.updatePieChart();
    },
    updatePieChart: function() {
      const ctx = document.getElementById('pieChart').getContext('2d');
      const data = {
        labels: ['Principal', 'Interest'],
        datasets: [{
          data: [this.loanAmount, this.interestPayable],
          backgroundColor: ['#007BFF', '#28A745']
        }]
      };
      const options = {
        responsive: true
      };
      new Chart(ctx, {
        type: 'pie',
        data: data,
        options: options
      });
    }
  }
});
</script>

</body>
</html>
