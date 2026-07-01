🔐 Lab: MFA & Phishing-Resistant Authentication — Microsoft Entra ID


Série: AZ-500 / SC-500 Hands-on Labs
Plataforma: Microsoft Entra ID
Duração: ~20 minutos
Nível: Intermédio




📋 Visão Geral

Este lab cobre a configuração completa de autenticação multifator no Microsoft Entra ID, desde o registo inicial de MFA até à implementação de autenticação phishing-resistant via Passkey (FIDO2). O objetivo é compreender a diferença entre "ter MFA" e "ter MFA verdadeiramente seguro".


🎯 Objetivos


 Configurar uma MFA Registration Policy via Identity Protection
 Criar uma Conditional Access Policy para exigir MFA em portais cloud
 Criar uma Authentication Strength personalizada com MFA phishing-resistant
 Ativar e registar um Passkey (FIDO2) via Microsoft Authenticator
 Verificar login com autenticação phishing-resistant end-to-end



🧠 Conceitos-Chave

MFA Registration Policy vs Conditional Access

São duas coisas distintas e frequentemente confundidas:

MFA Registration PolicyConditional AccessOnde viveIdentity ProtectionEntra ID > SecurityO que fazForça utilizadores a registar um método MFAExige o uso de MFA para aceder a recursosImpacto imediatoNenhum — utilizador ainda acede sem MFABloqueia acesso se MFA não for completadoQuando usarRollout gradual sem disrupçãoEnforcement ativo de políticas de segurança

Authentication Strength — porquê personalizar?

O Entra ID tem strengths pré-definidas ("Phishing-resistant MFA"), mas criar uma personalizada permite:


Escolher exatamente quais métodos são aceites (ex: Passkey FIDO2 + Microsoft Authenticator, excluindo SMS)
Adequar a requisitos regulatórios específicos (GDPR, DORA, NIS2)
Aplicar controlos diferentes por aplicação ou grupo de risco


Passkey (FIDO2) — o porquê

MFA tradicional (SMS, TOTP) ainda é vulnerável a ataques Adversary-in-the-Middle (AiTM):

[Utilizador] → [Site phishing (Evilginx)] → [Site real]
                  ↑ interceta token MFA

Um Passkey FIDO2 vincula a autenticação ao domínio legítimo criptograficamente. Um site falso nunca consegue completar o handshake — mesmo com a password roubada.


🔧 Tarefas Realizadas

Task 1 — MFA Registration Policy

Caminho: Entra ID → Protection → Identity Protection → MFA Registration Policy


Configurado para utilizador de teste (Delia Dennis)
Administradores excluídos via Exclude tab (break-glass account protection)
Policy definida como Enabled
Resultado: utilizador é forçado a registar o Microsoft Authenticator no próximo login, sem bloquear o acesso


Task 2 — Conditional Access: Require MFA for Admin Portals

Caminho: Entra ID → Security → Conditional Access → + New policy

Nome:          Require MFA for portals
Utilizadores:  Delia Dennis (incluída) | Administrator (excluído)
Target:        Microsoft Admin Portals
Grant:         Require multifactor authentication
Estado:        On

Verificação: Login no office.com não pede MFA. Login no entra.microsoft.com pede — Conditional Access aplicado por target resource, não por URL genérica.

Task 3 — Phishing-Resistant MFA com Passkey (FIDO2)

3.1 — Ativar o método:
Entra ID → Authentication methods → Policies → Passkey (FIDO2) → Enable

3.2 — Criar Authentication Strength personalizada:

Nome:        SC500 phishing resistant MFA
Métodos:     Passkeys (FIDO2) + Microsoft Authenticator

3.3 — Atualizar Conditional Access policy:


Grant: substituído Require MFA → Require authentication strength: SC500 phishing resistant MFA


3.4 — Registar Passkey via Bluetooth:


Requer ligação Bluetooth entre PC e telemóvel (VMs não suportam)
QR code gerado no portal → câmara do telemóvel → Passkey guardada no Microsoft Authenticator


3.5 — Login com Passkey:


Username → QR code no portal → câmara do telemóvel → biometria/PIN → autenticado



⚠️ Notas Técnicas


VMs não suportam Bluetooth — o registo de Passkey tem de ser feito no browser do PC físico, não dentro do ambiente de lab virtualizado
Authentication Strengths novas podem demorar alguns minutos a aparecer no dropdown do Conditional Access — fazer logout e re-login resolve
Excluir sempre contas de administrador e break-glass das políticas MFA restritivas — evita lock-out acidental do tenant



🔗 Referências


Microsoft Docs — MFA Registration Policy
Microsoft Docs — Conditional Access
Microsoft Docs — Authentication Strengths
Microsoft Docs — FIDO2 Passkeys
MITRE ATT&CK — T1111: MFA Interception



🗺️ Próximos Labs


 Privileged Identity Management (PIM) — Just-in-Time access para roles Entra e Azure
 Conditional Access — Self-Service Password Reset em ambientes híbridos
 Securing AI agents e declarative agents com API plugins


