
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Учет заправки</title>  
  <link rel="stylesheet" href="2tab.html">
  <style>
    body{font-family:Arial,Helvetica,sans-serif;background:#f5f5f5;margin:0;padding:20px}
    .container{max-width:1000px;margin:auto;background:#fff;padding:20px;border-radius:10px}
    h1,h2{text-align:center}
    .balances{display:flex;justify-content:space-around;flex-wrap:wrap}
    .balance-card{background:#fafafa;border:1px solid #ccc;border-radius:8px;padding:15px;margin:10px;flex:1 1 250px;text-align:center}
    .green{color:#27ae60}.red{color:#c0392b}
    .cards{display:flex;justify-content:space-around;flex-wrap:wrap;margin:20px 0}
    .card{border:1px solid #ccc;padding:15px;border-radius:8px;width:45%;min-width:280px;background:#fafafa;margin:10px}
    .card input,.card select,.container input{width:100%;padding:8px;margin:5px 0 10px}
    button{width:100%;padding:10px;background:#2980b9;color:#fff;border:none;cursor:pointer}
    button:hover{background:#1c5985}
    #historyList{list-style:none;padding:0}
    #historyList li{background:#eee;padding:8px;margin-bottom:5px;border-left:4px solid #2980b9;font-size:14px}
    .hint{text-align:center;color:#777;margin-top:10px}
  </style>
</head>
<body>
  <div class="container" id="login-screen">
    <h1>Авторизация</h1>
    <label>Логин:</label>
    <input type="text" id="login">
    <label>Пароль:</label>
    <input type="password" id="password">
    <button id="loginBtn">Войти</button>
    <p class="hint">Доступ: bilol / Imron / diyor — пароль 123</p>
  </div>

  <div class="container" id="main-app" style="display:none">
    <h1>Учёт заправки</h1>
    <div class="balances" id="balanceCards"></div>
    <div class="cards">
      <div class="card">
        <h2>Продажа топлива</h2>
        <label>Вид топлива</label>
        <select id="saleFuel">
          <option value="AI-95">АИ‑95</option>
          <option value="AI-90">АИ‑90</option>
          <option value="AI-80">АИ‑80</option>
        </select>
        <label>Количество (л)</label>
        <input type="number" id="saleLiters" min="0">
        <label>Цена за литр (сум)</label>
        <input type="number" id="salePrice" min="0">
        <button onclick="sellFuel()">Продать</button>
      </div>
      <div class="card">
        <h2>Закупка топлива</h2>
        <label>Вид топлива</label>
        <select id="buyFuel">
          <option value="AI-95">АИ‑95</option>
          <option value="AI-90">АИ‑90</option>
          <option value="AI-80">АИ‑80</option>
        </select>
        <label>Количество (л)</label>
        <input type="number" id="buyLiters" min="0">
        <label>Цена за литр (сум)</label>
        <input type="number" id="buyPrice" min="0">
        <button onclick="buyFuel()">Закупить</button>
      </div>
    </div>
    <div>
      <h2>История операций</h2>
      <ul id="historyList"></ul>
    </div>
  </div>

<script>
    
const users = {"bilol":"123","Imron":"123","diyor":"123"};
let currentUser = null;
let data = {
  cash: 0,
  fuels: {"AI-95":0,"AI-90":0,"AI-80":0},
  transactions: []
};

function loadData() {
  const saved = localStorage.getItem("fuelData");
  if (saved) data = JSON.parse(saved);
}

function saveData() {
  localStorage.setItem("fuelData", JSON.stringify(data));
}

loadData();

document.getElementById("loginBtn").addEventListener("click",()=>{
  const login=document.getElementById("login").value.trim();
  const pass=document.getElementById("password").value.trim();
  if(users[login]===pass){
    currentUser=login;
    document.getElementById("login-screen").style.display="none";
    document.getElementById("main-app").style.display="block";
    updateBalances();
    updateHistory();
  }else alert("Неверный логин или пароль!");
});

function updateBalances(){
  const balanceCards = document.getElementById("balanceCards");
  balanceCards.innerHTML="";
  const cashCard=document.createElement("div");
  cashCard.className="balance-card";
  cashCard.innerHTML=`<h3>Деньги</h3><p class="${data.cash<0?'red':'green'}">${data.cash.toFixed(2)} сум</p>`;
  balanceCards.appendChild(cashCard);
  for(const fuel in data.fuels){
    const card=document.createElement("div");
    card.className="balance-card";
    card.innerHTML=`<h3>${fuel}</h3><p class="${data.fuels[fuel]<0?'red':'green'}">${data.fuels[fuel].toFixed(2)} л</p>`;
    balanceCards.appendChild(card);
  }
}

function updateHistory(){
  const historyList = document.getElementById("historyList");
  historyList.innerHTML="";
  data.transactions.slice(-20).reverse().forEach(tx=>{
    const li=document.createElement("li");
    const sign=tx.type==='sale'?'+':'-';
    li.textContent=`👤 ${tx.user} → ${tx.type==='sale'?'Продажа':'Закупка'} ${tx.fuel}: ${tx.liters}л по ${tx.price} = ${sign}${tx.amount} | Cash: ${tx.cash.toFixed(2)} | ${tx.fuel}: ${tx.fuelBalance.toFixed(2)}`;
    historyList.appendChild(li);
  });
}

function confirmCode(){
  const code=prompt("Введите код для подтверждения (123):");
  return code==="123";
}

function sellFuel(){
  if(!confirmCode()) return alert("Неверный код!");
  const fuel=document.getElementById("saleFuel").value;
  const liters=parseFloat(document.getElementById("saleLiters").value);
  const price=parseFloat(document.getElementById("salePrice").value);
  if(liters<=0||price<=0||isNaN(liters)||isNaN(price)) return alert("Корректные значения!");
  const amount=liters*price;
  data.fuels[fuel]-=liters;
  data.cash+=amount;
  data.transactions.push({type:'sale',user:currentUser,fuel,liters,price,amount,cash:data.cash,fuelBalance:data.fuels[fuel]});
  updateBalances();
  updateHistory();
  saveData();
  document.getElementById("saleLiters").value='';
  document.getElementById("salePrice").value='';
}

function buyFuel(){
  if(!confirmCode()) return alert("Неверный код!");
  const fuel=document.getElementById("buyFuel").value;
  const liters=parseFloat(document.getElementById("buyLiters").value);
  const price=parseFloat(document.getElementById("buyPrice").value);
  if(liters<=0||price<=0||isNaN(liters)||isNaN(price)) return alert("Корректные значения!");
  const amount=liters*price;
  data.fuels[fuel]+=liters;
  data.cash-=amount;
  data.transactions.push({type:'purchase',user:currentUser,fuel,liters,price,amount,cash:data.cash,fuelBalance:data.fuels[fuel]});
  updateBalances();
  updateHistory();
  saveData();
  document.getElementById("buyLiters").value='';
  document.getElementById("buyPrice").value='';
}
</script>
</body>
</html>
