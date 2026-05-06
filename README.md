<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>השוברים שלנו</title>
    
    <!-- חיבור לספריות Firebase בגרסה תואמת לדפדפן -->
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database-compat.js"></script>
    
    <style>
        body { 
            font-family: -apple-system, system-ui, sans-serif; 
            background-color: #f2f2f7; 
            margin: 0; 
            padding: 20px; 
            color: #1c1c1e; 
            padding-bottom: 100px;
        }
        h1 { text-align: center; color: #007aff; margin-bottom: 30px; }
        .card { 
            background: white; 
            border-radius: 15px; 
            padding: 20px; 
            margin-bottom: 20px; 
            box-shadow: 0 4px 12px rgba(0,0,0,0.08); 
            border: 1px solid #d1d1d6; 
        }
        .voucher-header { 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            margin-bottom: 15px;
        }
        .voucher-name { font-size: 1.3rem; font-weight: 700; }
        .balance { color: #34c759; font-weight: 800; font-size: 1.4rem; }
        .code-area { 
            background: #f2f2f7; 
            padding: 15px; 
            border-radius: 12px; 
            text-align: center; 
            font-family: 'Courier New', monospace; 
            font-size: 1.6rem; 
            letter-spacing: 3px; 
            font-weight: bold;
            border: 1px dashed #8e8e93;
        }
        .actions { 
            display: flex; 
            gap: 15px; 
            margin-top: 20px; 
        }
        button { 
            flex: 1;
            padding: 12px;
            border-radius: 10px;
            border: none;
            font-weight: 600;
            font-size: 1rem;
            cursor: pointer;
        }
        .btn-update { background-color: #007aff; color: white; }
        .btn-delete { background-color: #ff3b30; color: white; opacity: 0.8; }
        .add-btn { 
            background-color: #007aff; 
            color: white; 
            border: none; 
            width: 65px; 
            height: 65px; 
            border-radius: 32.5px; 
            position: fixed; 
            bottom: 30px; 
            left: 30px; 
            font-size: 30px; 
            box-shadow: 0 4px 15px rgba(0,122,255,0.4);
            display: flex;
            align-items: center;
            justify-content: center;
        }
    </style>
</head>
<body>

    <h1>השוברים שלנו 🎫</h1>
    <div id="vouchers-list">טוען נתונים...</div>
    
    <button class="add-btn" onclick="addNewVoucher()">+</button>

    <script>
        // פרטי ה-Firebase שלך משולבים כאן
        const firebaseConfig = {
            apiKey: "AIzaSyCkiBI4o1so5_XQc72Xxg-V5cYJcutpuZY",
            authDomain: "family-vouchers-301e3.firebaseapp.com",
            projectId: "family-vouchers-301e3",
            databaseURL: "https://family-vouchers-301e3-default-rtdb.europe-west1.firebasedatabase.app",
            storageBucket: "family-vouchers-301e3.firebasestorage.app",
            messagingSenderId: "242043746201",
            appId: "1:242043746201:web:b003f572f65136304ac508"
        };

        // אתחול המערכת
        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        // האזנה לסנכרון בזמן אמת
        database.ref('vouchers').on('value', (snapshot) => {
            const data = snapshot.val();
            renderVouchers(data);
        });

        function renderVouchers(data) {
            const container = document.getElementById('vouchers-list');
            container.innerHTML = '';
            
            if (!data) {
                container.innerHTML = '<p style="text-align:center; color:#8e8e93;">אין שוברים כרגע. לחצו על ה-+ כדי להוסיף.</p>';
                return;
            }

            Object.keys(data).forEach(id => {
                const v = data[id];
                container.innerHTML += `
                    <div class="card">
                        <div class="voucher-header">
                            <span class="voucher-name">${v.name}</span>
                            <span class="balance">₪${v.balance}</span>
                        </div>
                        <div class="code-area">${v.code}</div>
                        <div class="actions">
                            <button class="btn-update" onclick="updateVoucher('${id}', ${v.balance})">עדכן יתרה</button>
                            <button class="btn-delete" onclick="deleteVoucher('${id}')">מחק</button>
                        </div>
                    </div>
                `;
            });
        }

        function addNewVoucher() {
            const name = prompt("שם השובר (לדוגמה: ביימי מסעדות):");
            const balance = prompt("מה הסכום שיש בכרטיס?");
            const code = prompt("מה מספר השובר / קוד המימוש?");
            
            if (name && balance && code) {
                database.ref('vouchers').push({
                    name: name,
                    balance: parseFloat(balance),
                    code: code
                });
            }
        }

        function updateVoucher(id, currentBalance) {
            const amountUsed = prompt(`יתרה נוכחית: ₪${currentBalance}\nכמה שילמת מהשובר?`);
            if (amountUsed && !isNaN(amountUsed)) {
                const newBalance = currentBalance - parseFloat(amountUsed);
                database.ref('vouchers/' + id).update({ balance: newBalance });
            }
        }

        function deleteVoucher(id) {
            if (confirm("בטוח שרוצים למחוק את השובר מהרשימה?")) {
                database.ref('vouchers/' + id).remove();
            }
        }
    </script>
</body>
</html>
