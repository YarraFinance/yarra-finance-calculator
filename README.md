<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta http-equiv="X-UA-Compatible" content="IE=edge" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Yarra Finance Loan Repayment Calculator</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background-color: #d1d5db;
    display: flex;
    justify-content: center;
    padding: 20px;
    margin: 0;
  }
  .container {
    background: #e5e7eb;
    padding: 24px;
    border-radius: 16px;
    max-width: 400px;
    width: 100%;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    border: 1px solid black;
  }
  h2 { color: #dc2626; text-align: center; padding: 8px; border-radius: 8px; }
  label { display: block; margin-bottom: 4px; color: #4181ff; font-weight: bold; }
  input, select { width: 100%; padding: 8px; margin-bottom: 16px; border: 1px solid #9ca3af; border-radius: 6px; }
  input[type=range] { accent-color: #4181ff; }
  .note { font-size: 12px; color: #4b5563; margin-bottom: 8px; }
  .result { background: white; padding: 16px; border-radius: 8px; margin-top: 16px; text-align: center; color: #4181ff; }
  .slider-container { margin-bottom: 16px; }
  .enquire-button { background-color: #4181ff; color: #ffffff; padding: 12px; border-radius: 6px; text-align: center; display: block; margin: 16px auto 0; text-decoration: none; font-weight: bold; cursor: pointer; border: none; }
</style>
</head>

<body>
<div class="container">
  <h2><span style="font-family:Georgia;color:#4181ff;text-decoration:underline;">yarra</span><span style="font-family:Arial;color:#808080;">finance</span></h2>
  <p class="note">*Minimum Loan Amount for Low rate is $20,000. Rate increases below minimum amount.</p>

  <label>Borrowing Amount ($)</label>
  <input type="text" id="amount" value="50000"/>

  <label>Deposit ($)</label>
  <input type="text" id="deposit" value="0"/>

  <label>Term (months)</label>
  <select id="term">
    <option value="36">36 months (3 years)</option>
    <option value="48">48 months (4 years)</option>
    <option value="60">60 months (5 years)</option>
  </select>

  <div class="slider-container">
    <label>Balloon Amount: <span id="balloonValue">0%</span></label>
    <input type="range" id="balloonSlider" min="0" max="50" value="0" />
  </div>

  <p class="note" style="text-align:center; color:#4181ff;">We will beat any written quote where possible.</p>
  <p class="note">*Payments are in Advance (<strong>Arrears Available</strong>).</p>

  <div class="result">
    <p>Payments from: <strong id="repayment">$0.00</strong></p>
    <p class="note">Subject to approval criteria</p>
  </div>

  <button class="enquire-button" id="enquireBtn">Refer Customer</button>
</div>

<script>
const baseRate = 0.0678;
const highRate = 0.095;
const minLoanAmount = 20000;

const amountInput = document.getElementById('amount');
const depositInput = document.getElementById('deposit');
const termSelect = document.getElementById('term');
const balloonSlider = document.getElementById('balloonSlider');
const balloonValue = document.getElementById('balloonValue');
const repaymentDisplay = document.getElementById('repayment');

function getMaxBalloon(term) {
  if (term == 36) return 50;
  if (term == 48) return 40;
  if (term == 60) return 30;
  return 0;
}

function formatNumber(num) { return num.toLocaleString(); }
function parseNumber(str) { return Number(str.replace(/,/g, '')) || 0; }

function updateSliderMax() {
  const term = parseInt(termSelect.value);
  balloonSlider.max = getMaxBalloon(term);
  if (parseInt(balloonSlider.value) > balloonSlider.max) balloonSlider.value = balloonSlider.max;
  balloonValue.textContent = balloonSlider.value + '%';
}

function calculateRepayment() {
  let amount = parseNumber(amountInput.value);
  let deposit = parseNumber(depositInput.value);
  const term = parseInt(termSelect.value);
  const balloonPercent = parseInt(balloonSlider.value)/100;

  const rate = amount < minLoanAmount ? highRate : baseRate;
  const months = term;
  const monthlyRate = rate/12;
  const netAmount = amount - deposit;
  const balloonAmount = netAmount * balloonPercent;
  const financedAmount = netAmount - balloonAmount / Math.pow(1 + monthlyRate, months);
  const repayment = (financedAmount * monthlyRate) / (1 - Math.pow(1 + monthlyRate, -months));

  repaymentDisplay.textContent = '$' + repayment.toFixed(2);
  amountInput.value = formatNumber(amount);
  depositInput.value = formatNumber(deposit);
  balloonValue.textContent = Math.round(balloonPercent*100) + '%';
}

amountInput.addEventListener('blur', calculateRepayment);
depositInput.addEventListener('blur', calculateRepayment);
termSelect.addEventListener('change', () => { updateSliderMax(); calculateRepayment(); });
balloonSlider.addEventListener('input', calculateRepayment);

updateSliderMax();
calculateRepayment();

// Email auto-populate
document.getElementById('enquireBtn').addEventListener('click', () => {
  const amount = amountInput.value;
  const deposit = depositInput.value;
  const balloon = balloonSlider.value;
  const term = termSelect.value;
  const repayment = repaymentDisplay.textContent;

  const subject = encodeURIComponent("New Finance Referral");

  const bodyText = `==================== YARRA FINANCE REFERRAL ==================

Borrowing Amount:       $${amount}
Deposit Amount:         $${deposit}
Balloon Amount:         ${balloon}%
Term:                   ${term} months
Estimated Repayment:    ${repayment}
==============================================================
*WHICH COMPANY ARE YOU REFFERING THIS CUSTOMER FROM?* [ ____ ]
==============================================================

CUSTOMER INFORMATION (Please Fill out as much as you can)

*Customer First Name: 
*Customer Surname: 
*Customer Mobile: 
*Customer Email: 
*Asset Details: 
*Customer Company Name: 
*Customer Company ABN: 

Additional Notes:

`;

  const body = encodeURIComponent(bodyText);
  window.location.href = `mailto:admin@yarrafinance.com?subject=${subject}&body=${body}`;
});
</script>
</body>
</html>
