# Lösning: Strawberry Secret Vault

## Krypteringsmetod

Applikationen använder **symmetrisk kryptering** med AES-256-GCM.

Det går att se på flera ställen i Program.cs:

1. På rad 68 skapas nyckeln med **PBKDF2**. Samma lösenord och salt används för att skapa samma nyckel igen.
2. På rad 72 används **klassen AesGcm**, som använder samma nyckel för både kryptering och dekryptering.
3. På rad 76 används **Encrypt-metoden**, där meddelandet krypteras med hjälp av nyckeln och en nonce (En nonce är ett unikt värde som används vid kryptering för att samma meddelande inte ska ge samma krypterade resultat varje gång.).

Det som visar att krypteringen är symmetrisk är alltså att samma nyckel används åt båda hållen. Vid asymmetrisk kryptering hade man i stället använt två olika nycklar, en publik och en privat.

---

## Arbetsgång

1. Jag började med att **Läsa konfigurationen** i `appsettings.json`. På rad 11 hittade jag den hemliga frasen som var base64-kodad: 
`"SecretPhraseB64": "a3VyZGlza2Fyw6R2ZW4="`

2. Jag **sökte på Google** efter "powershell decode base64 svenska" för att ta reda på hur jag kunde avkoda strängen (dekoda).

3. Sedan körde jag följade kommando, dvs **dekoda**, i
   ```powershell
   [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("a3VyZGlza2Fyw6R2ZW4="))
   ```

4. I Program.cs på rad 31–32 såg jag att de krypterade meddelandena skulle ligga i mappen data/<timestamp>/. Jag **flyttade därför mappen** 20250826_210423714 dit.

5. Därefter **startade jag applikationen** genom att köra
dotnet run 
från projektroot

6. När menyn visades skrev jag in den dekodade frasen från steg 3 för att **låsa upp dekrypteringen**.

7. Applikationen visade sedan de tillgängliga krypterade meddelandena. **Jag valde meddelande**: [1] 20250826_210423714

8. Efter det **visades det dekrypterade meddelandet** på skärmen.

---

## Meddelandet

Grattis, ni har lyckats dekryptera meddelandet! 🎉 Det här visar att ni kan analysera kod, hitta dolda funktioner och förstå hur kryptering fungerar i praktiken. 💻🔐 Kom ihåg: Riktig säkerhet bygger aldrig på att gömma koden – utan på stark kryptering, korrekt nyckelhantering och god design.

---

## Motivering

Applikationen gör ett allvarligt kryptografiskt fel eftersom lösenordet sparas tillsammans med det krypterade meddelandet i Program.cs på rad 99. Lösenordet är bara base64-kodat, och base64 är inte kryptering utan endast ett sätt att koda information. Det gör att lösenordet enkelt kan avkodas och användas för att läsa meddelandet. Detta bryter principen om "defense in depth" och felet hör därför till A04 Cryptographic Failures i OWASP Top 10:2025, eftersom ett hemligt värde lagras utan tillräckligt skydd. För att stoppa detta borde raden File.WriteAllText(pwdPath, Convert.ToBase64String(...)) tas bort och ersättas med att användaren själv får skriva in lösenordet när meddelandet ska dekrypteras.
