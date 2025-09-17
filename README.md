<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Event Management - SSS Radio</title>
  <style>
    body { font-family: Arial; background: #f4f8fc; margin: 0; padding: 20px; }
    .container { max-width: 800px; margin:auto; background:#fff; padding:20px; border-radius:12px; box-shadow:0 4px 12px rgba(0,0,0,0.1);}
    h1 { text-align:center; color:#0073e6; margin-bottom:20px;}
    label {display:block; margin:12px 0 6px; font-weight:bold;}
    input {width:100%; padding:8px; margin-bottom:15px; border:1px solid #ccc; border-radius:6px;}
    button {background:#0073e6; color:white; padding:10px; border:none; border-radius:6px; cursor:pointer;}
    button:hover {background:#005bb5;}
    .summary {margin-top:20px; padding:15px; background:#f0f8ff; border-radius:8px; display:none;}
    .qr-section {margin-top:20px; text-align:center; display:none;}
    .qr-section a {display:inline-block; margin-top:10px; background:#00b894; color:white; padding:8px 14px; border-radius:6px; text-decoration:none;}
    footer {margin-top:30px; text-align:center; font-size:14px; color:#555;}
  </style>
</head>
<body>
  <div class="container">
    <h1>🎶 SSS Radio Set Management</h1>
    <form id="orderForm">
      <label>Event Name:</label>
      <input type="text" id="eventName" required>
      <label>Full Name:</label>
      <input type="text" id="name" required>
      <label>Phone Number:</label>
      <input type="tel" id="phone" required placeholder="+91 9876543210">
      <label>Email:</label>
      <input type="email" id="email" required placeholder="example@email.com">
      <label>Chairs:</label><input type="number" id="chairs" min="0">
      <label>Tables:</label><input type="number" id="tables" min="0">
      <label>Zoomers:</label><input type="number" id="zoomers" min="0">
      <label>Speakers:</label><input type="number" id="speakers" min="0">
      <label>Lights:</label><input type="number" id="lights" min="0">
      <label>Cooking Materials:</label><input type="number" id="cooking" min="0">
      <label>Advance Payment (₹):</label><input type="number" id="advance" min="0" required>
      <button type="button" onclick="submitOrder()">Submit Order</button>
    </form>

    <div class="summary" id="summary"></div>
    <div class="qr-section" id="payment">
      <h3>💳 Advance Payment</h3>
      <p>Scan QR or click the button:</p>
      <img id="qr" width="200" height="200">
      <br>
      <a id="upiLink" target="_blank">Pay Advance</a>
    </div>
  </div>

  <footer>📞 Contact: +91 9789750637 | 📍 Dharmapuri, Tamil Nadu</footer>

  <script>
    function submitOrder() {
      const data = {
        eventName: document.getElementById("eventName").value,
        name: document.getElementById("name").value,
        phone: document.getElementById("phone").value,
        email: document.getElementById("email").value,
        chairs: parseInt(document.getElementById("chairs").value||0),
        tables: parseInt(document.getElementById("tables").value||0),
        zoomers: parseInt(document.getElementById("zoomers").value||0),
        speakers: parseInt(document.getElementById("speakers").value||0),
        lights: parseInt(document.getElementById("lights").value||0),
        cooking: parseInt(document.getElementById("cooking").value||0),
        advancePayment: parseInt(document.getElementById("advance").value||0),
      };

      const prices = { chairs:10, tables:50, zoomers:2000, speakers:1250, lights:150, cooking:5000 };
      data.totalCost = data.chairs*prices.chairs + data.tables*prices.tables + data.zoomers*prices.zoomers +
                       data.speakers*prices.speakers + data.lights*prices.lights + data.cooking*prices.cooking;
      data.remainingPayment = data.totalCost - data.advancePayment;

      if(data.advancePayment<=0){ alert("⚠️ Enter valid advance amount"); return;}

      // Show summary
      const summaryDiv = document.getElementById("summary");
      summaryDiv.style.display = "block";
      summaryDiv.innerHTML = `
        <h3>✅ Order Confirmed</h3>
        <p><strong>Event:</strong> ${data.eventName}</p>
        <p><strong>Name:</strong> ${data.name}</p>
        <p><strong>Phone:</strong> ${data.phone}</p>
        <p><strong>Email:</strong> ${data.email}</p>
        <hr>
        <p><strong>Total:</strong> ₹${data.totalCost}</p>
        <p><strong>Advance:</strong> ₹${data.advancePayment}</p>
        <p><strong>Remaining:</strong> ₹${data.remainingPayment}</p>
      `;

      // UPI QR code
      const upiId = "venkataadhav9867@oksbi";
      const payUrl = `upi://pay?pa=${upiId}&pn=${encodeURIComponent(data.name)}&am=${data.advancePayment}&cu=INR`;
      document.getElementById("qr").src = `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${encodeURIComponent(payUrl)}`;
      document.getElementById("upiLink").href = payUrl;
      document.getElementById("upiLink").innerText = `Pay Advance ₹${data.advancePayment}`;
      document.getElementById("payment").style.display = "block";

      // Send SMS to Event Management
      fetch("/send-sms", {
        method:"POST",
        headers:{"Content-Type":"application/json"},
        body: JSON.stringify(data)
      })
      .then(res=>res.json())
      .then(res=>{
        if(res.success){ alert("✅ SMS sent to Event Management number"); }
        else{ alert("❌ SMS failed: "+res.error);}
      })
      .catch(err=>alert("❌ Error sending SMS: "+err));
    }
  </script>
</body>
</html>
