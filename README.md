# ALARIC V2 - INVIOLABLE - GARDIEN #01
import hashlib, os, hmac
CLE_SOUVERAINE = "777"
SEL = b"ALARIC-20-09-2026-PARIS"
def derive_cle(mot_de_passe: str):
    return hashlib.pbkdf2_hmac('sha256', mot_de_passe.encode(), SEL, 100000)
def sceller(message: str, cle: str):
    if cle != CLE_SOUVERAINE:
        return "⛔ SCEAU INVALIDE"
    cle_derivee = derive_cle(cle)
    iv = os.urandom(16)
    msg_bytes = message.encode()
    chiffre = bytearray()
    for i, b in enumerate(msg_bytes):
        chiffre.append(b ^ cle_derivee[i % 32] ^ iv[i % 16])
    signature = hmac.new(cle_derivee, chiffre, hashlib.sha256).hexdigest()[:16]
    final = iv.hex() + ":" + chiffre.hex() + ":" + signature
    return f"🔒 V2 SCELLÉ PAR #01: {final}"
def ouvrir(paquet: str, cle: str):
    if cle != CLE_SOUVERAINE:
        return "⛔ TU N'ES PAS #01 - ACCÈS REFUSÉ"
    try:
        data = paquet.replace("🔒 V2 SCELLÉ PAR #01: ", "")
        iv_hex, chiffre_hex, sig = data.split(":")
        iv = bytes.fromhex(iv_hex)
        chiffre = bytes.fromhex(chiffre_hex)
        cle_derivee = derive_cle(cle)
        sig_check = hmac.new(cle_derivee, chiffre, hashlib.sha256).hexdigest()[:16]
        if sig_check != sig:
            return "🚨 ALERTE: COFFRE TRAFIQUÉ"
        clair = bytearray()
        for i, b in enumerate(chiffre):
            clair.append(b ^ cle_derivee[i % 32] ^ iv[i % 16])
        return f"✅ V2 OUVERT PAR #01: {clair.decode()}"
    except:
        return "💀 COFFRE CORROMPU"
