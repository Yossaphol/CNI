# ใส่ค่าลงในตารางด้านล่าง และ configure อุปกรณ์ ให้ถูกต้องครบ ถ้วน ตามข้อกำหนดต่อไปนี้


1. กำหนดให้ Subnet IPv4 ที่เชื่อมต่อกันระหว่าง Router เป็น [192.168.1.176/28]

    • ให้นักศึกษาแบ่ง Subnet เป็น 2 Subnets โดย Assign ให้กับ Network ระหว่าง Router แบ่งตามลำดับเท่านั้น
    • Subnet ระหว่าง CORE-ROUTER และ MAIN-ROUTER
    • มี IP ที่สามารถใช้งานได้ 2 IP Address และต้องใกล้เคียงมากที่สุด
    • CORE-ROUTER มี IPV4 Address โดยใช้ Host IP ลำดับที่ 1
    • MAIN-ROUTER มี IPV4 Address โดยใช้ Host IP ลำดับสุดท้าย
    • กำหนดให้ Default Gateway ของ MAIN-ROUTER ส่งมาที่ CORE-ROUTER
    • Subnet ระหว่าง MAIN-ROUTER และ SUB-ROUTER
    • มี IP ที่สามารถใช้งานได้ 2 IP Address และต้องใกล้เคียงมากที่สุด
    • MAIN-ROUTER มี IPV4 Address โดยใช้ Host IP ลำดับที่ 1
    • SUB-ROUTER มี IPV4 Address โดยใช้ Host IP ลำดับสุดท้าย
    • กำหนดให้ Default Gateway ของ SUB-ROUTER ส่งมาที่ MAIN-ROUTE

---