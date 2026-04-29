# Contact Us

<div id="contact-form-container"></div>

<script>
(function(){
var c=document.getElementById('contact-form-container');
if(!c)return;
c.innerHTML=`
<div style="max-width:520px">
<p style="font-size:15px;color:var(--t2);margin-bottom:24px">Have questions about the AutoShares API? Need help with your integration? Our team is here to help.</p>

<form id="contact-form" onsubmit="return handleContact(event)" style="display:flex;flex-direction:column;gap:16px">

<div style="display:grid;grid-template-columns:1fr 1fr;gap:12px">
  <div style="display:flex;flex-direction:column;gap:4px">
    <label style="font-size:12px;font-weight:600;color:var(--t4);text-transform:uppercase;letter-spacing:.05em">First Name *</label>
    <input id="ct-first" required style="padding:10px 14px;border:1px solid var(--b1);border-radius:8px;background:var(--bg);color:var(--t1);font-size:14px;font-family:inherit" placeholder="Jane">
  </div>
  <div style="display:flex;flex-direction:column;gap:4px">
    <label style="font-size:12px;font-weight:600;color:var(--t4);text-transform:uppercase;letter-spacing:.05em">Last Name *</label>
    <input id="ct-last" required style="padding:10px 14px;border:1px solid var(--b1);border-radius:8px;background:var(--bg);color:var(--t1);font-size:14px;font-family:inherit" placeholder="Smith">
  </div>
</div>

<div style="display:flex;flex-direction:column;gap:4px">
  <label style="font-size:12px;font-weight:600;color:var(--t4);text-transform:uppercase;letter-spacing:.05em">Email Address *</label>
  <input id="ct-email" type="email" required style="padding:10px 14px;border:1px solid var(--b1);border-radius:8px;background:var(--bg);color:var(--t1);font-size:14px;font-family:inherit" placeholder="jane@acmetrading.com">
</div>

<div style="display:flex;flex-direction:column;gap:4px">
  <label style="font-size:12px;font-weight:600;color:var(--t4);text-transform:uppercase;letter-spacing:.05em">Company / Organization</label>
  <input id="ct-company" style="padding:10px 14px;border:1px solid var(--b1);border-radius:8px;background:var(--bg);color:var(--t1);font-size:14px;font-family:inherit" placeholder="Acme Trading Inc.">
</div>

<div style="display:flex;flex-direction:column;gap:4px">
  <label style="font-size:12px;font-weight:600;color:var(--t4);text-transform:uppercase;letter-spacing:.05em">Topic *</label>
  <select id="ct-topic" required style="padding:10px 14px;border:1px solid var(--b1);border-radius:8px;background:var(--bg);color:var(--t1);font-size:14px;font-family:inherit;cursor:pointer">
    <option value="">Select a topic...</option>
    <option value="api-access">API Access & Onboarding</option>
    <option value="technical">Technical Support</option>
    <option value="integration">Integration Help</option>
    <option value="production">Production Upgrade</option>
    <option value="back-office">Back Office API Access</option>
    <option value="market-data">Market Data & Entitlements</option>
    <option value="partnership">Partnership Inquiry</option>
    <option value="billing">Billing & Pricing</option>
    <option value="bug">Bug Report</option>
    <option value="feature">Feature Request</option>
    <option value="other">Other</option>
  </select>
</div>

<div style="display:flex;flex-direction:column;gap:4px">
  <label style="font-size:12px;font-weight:600;color:var(--t4);text-transform:uppercase;letter-spacing:.05em">Message *</label>
  <textarea id="ct-message" required rows="5" style="padding:10px 14px;border:1px solid var(--b1);border-radius:8px;background:var(--bg);color:var(--t1);font-size:14px;font-family:inherit;resize:vertical" placeholder="Describe how we can help..."></textarea>
</div>

<button type="submit" id="ct-submit" style="padding:12px 24px;background:var(--ac);color:#fff;border:0;border-radius:10px;font-size:15px;font-weight:700;cursor:pointer;font-family:inherit;transition:all .12s">Send Message</button>

<div id="ct-status" style="display:none;padding:14px 18px;border-radius:10px;font-size:13px;line-height:1.6"></div>

</form>
</div>
`;
var style=document.createElement('style');
style.textContent='#contact-form input:focus,#contact-form select:focus,#contact-form textarea:focus{border-color:var(--ac);outline:none;box-shadow:0 0 0 3px var(--ac-dim)} #ct-submit:hover{filter:brightness(1.08)} #ct-submit:disabled{opacity:.5;cursor:not-allowed}';
document.head.appendChild(style);
})();
</script>

## Other Ways to Reach Us

| Channel | Details |
|---------|---------|
| **Email** | support@autoshares.com |
| **Sales** | sales@autoshares.com |
| **Website** | [autoshares.com](https://autoshares.com) |

## Response Times

| Topic | Typical Response |
|-------|-----------------|
| API Access & Onboarding | Same business day |
| Technical Support | Within 4 hours |
| Production Upgrade | 1-2 business days |
| Partnership Inquiry | 1-2 business days |
| Bug Reports | Within 24 hours |

---

&copy; 2026 AutoShares Fintech Solutions, LLC. All rights reserved.
