# Richtlinie 3.3.1
``` JS
<script>
  document.addEventListener("DOMContentLoaded", function () {
    const form = document.querySelector("form");
    if (!form) return;
    form.setAttribute("novalidate", true);
    
    const fields = form.querySelectorAll("input, textarea");
    //Hier eigene Nachrichten für deine Elemente eintragen und die Elementnamen anpassen
    const customMessages = {
      "name": "Bitte Ihren Namen eingeben",
      "email": "Bitte eine gültige E-Mail-Adresse eingeben (z. B. max.mustermann@mailservice.com)",
      "message" : "Bitte eine Nachricht eingeben.",
      "address" : "Bitte eine Adresse eingeben (Format: Musterstr. 7)",
      "city" : "Bitte eine Stadt eingeben.",
      "cardnumber" : "Bitte eine gültige Kreditkartennummer eingeben (Format: 1234 5678 9123 4567)",
      "ccv" : "Bitte die 3-stellige Sicherheitsnummer auf der Rückseite ihrer Kreditkarte eingeben (z. B. 557)",
      "day" : "Bitte einen Tag als Nummer eingeben (nicht Montag sondern z. B. 3)",
      "year" : "Bitte eine 4-stellige Jahreszahl eingeben (z. B. 2001)",
      "password" : "Bitte ein Passwort eingeben",
      "password2" : "Bite das gleiche Passwort eingeben"
    };
      
    function showError(field, message) {
      const wrapper = field.closest(".form-group"); //Falls anderer Klassenname des unterordnenden Divs genutzt, hier ändern
      const error = wrapper?.querySelector(".error-message"); //Falls anderen Klassennamen des Fehlermeldungs-Divs genutzt, hier ändern
      if (error) {
        error.textContent = message;
        error.style.display = "block";
      }
      field.classList.add("invalid");
    }

    function clearError(field) {
      const wrapper = field.closest(".form-group");
      const error = wrapper?.querySelector(".error-message");
      if (error) {
        error.textContent = "";
        error.style.display = "none";
      }
      field.classList.remove("invalid");
    }
    
    function validateField(field) {
      const name = field.getAttribute("name");
      const value = field.value.trim();
      let valid = true;
      if (field.hasAttribute("required") && value === "") {
        showError(field, customMessages[name] || "Bitte füllen Sie dieses Feld aus.");
        valid = false;
      } else if (field.type === "email" && value !== "") {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(value)) {
          showError(field, customMessages["email"]);
          valid = false;
        } else {
          clearError(field);
        }
      } else {
        clearError(field);
      }
      return valid;
    }
    
    fields.forEach(field => {
      field.addEventListener("blur", () => validateField(field));
    });
    
    form.addEventListener("submit", function (e) {
      e.preventDefault();
      let allValid = true;
      fields.forEach(field => {
        const isValid = validateField(field);
        if (!isValid) {
          allValid = false;
        }
      });
      if (allValid) {
        form.submit();
      }
    });
  });
</script>
```

# Richtlinie 3.3.3
```JS
<script>
  document.addEventListener("DOMContentLoaded", () => {
    const emailInput = document.getElementById("email"); //Hier den Namen deines E-Mail Feldes eintragen
    const suggestionBox = document.getElementById("email-suggestion"); //Hier den Namen deines Vorschlags-divs eintragen
    const domains = ["gmail.com", "hotmail.com", "yahoo.com", "outlook.com", "web.de", "gmx.de"];
    
    function levenshtein(a, b) {
      const matrix = Array.from({length: b.length + 1}, () => Array(a.length + 1).fill(0));
      
      for(let i=0; i<=a.length; i++) {
        matrix[0][i] = i;
      }
      for(let j=0; j<=b.length; j++) {
        matrix[j][0] = j;
      }
      for(let j=1; j<=b.length; j++) {
        for(let i=1; i<=a.length; i++) {
          if(a[i-1] === b[j-1]) {
            matrix[j][i] = matrix[j-1][i-1];
          } else {
          matrix[j][i] = Math.min(matrix[j-1][i-1]+1, matrix[j][i-1]+1, matrix[j-1][i]+1);
          }
        }
      }
      return matrix[b.length][a.length];
    }
    
    emailInput.addEventListener("input", () => {
      const val = emailInput.value;
      suggestionBox.style.display = "none";
      const atIndex = val.indexOf("@");
      if (atIndex === -1) {
        return;
      }
      const localPart = val.slice(0, atIndex);
      const domainPart = val.slice(atIndex + 1).toLowerCase();
      if (!domainPart) return;
      let bestMatch = null;
      let bestDistance = Infinity;
      
      for (const domain of domains) {
        const dist = levenshtein(domainPart, domain);
        if (dist > 0 && dist < bestDistance && dist <= 2) {
          bestDistance = dist;
          bestMatch = domain;
            }
      }
      
      if (bestMatch) {
      const suggestion = `${localPart}@${bestMatch}`;
      suggestionBox.textContent = `Meinten Sie: ${suggestion} ?`;
      suggestionBox.style.display = "block";
      }
    });
    
    suggestionBox.addEventListener("click", () => {
      const text = suggestionBox.textContent;
      const suggestedEmail = text.match(/Meinten Sie: (.+) \?/)[1];
      emailInput.value = suggestedEmail;
      suggestionBox.style.display = "none";
      emailInput.focus();
      });

    emailInput.addEventListener("blur", () => {
      setTimeout(() => suggestionBox.style.display = "none", 200);
    });
  });
</script>
```