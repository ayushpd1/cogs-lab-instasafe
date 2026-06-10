# Lab 2.2 – Keycloak SAML SSO Configuration

## Objective

Deploy Keycloak as an Identity Provider (IdP), create users and groups, configure a SAML client, and validate a basic SAML authentication workflow.

---

# Environment

- Platform: AWS EC2
- Operating System: Ubuntu
- Keycloak Version: 23.0.0
- Public IP: 44.222.176.144

---

# Evidence

## 1. Keycloak Realm, User and SAML Client

![Keycloak Admin Console](./images/keycloak_admin_console.png)

The realm **instasafe-lab** was created successfully.

Configured resources:

- Realm: `instasafe-lab`
- User: `testuser`
- Group: `support-team`
- SAML Client: `InstaSafe SP Simulation`

---

## 2. SAML Client Configuration

![SAML Client Configuration](./images/saml_client_settings.png)

### Client Details

| Setting | Value |
|----------|----------|
| Client Type | SAML |
| Client ID | https://sp.instasafe.local/saml |
| Valid Redirect URI | http://44.222.176.144:9090/saml/callback |
| Master SAML Processing URL | http://44.222.176.144:9090/saml/callback |

---

## 3. Attribute Mappers

![Attribute Mappers](./images/saml_attribute_mappers.png)

### Email Mapper

Purpose:

Passes the user's email address inside the SAML assertion.

Configuration:

- Mapper Type: User Property
- User Property: email
- SAML Attribute Name: email

### Groups Mapper

Purpose:

Passes Keycloak group membership information to the Service Provider.

Configuration:

- Mapper Type: Group List
- SAML Attribute Name: groups

---

## 4. IdP Metadata XML

![IdP Metadata](./images/idp_metadata_xml.png)

Metadata downloaded from:

```text
http://44.222.176.144:8080/realms/instasafe-lab/protocol/saml/descriptor
```

### Entity ID

```xml
entityID="http://44.222.176.144:8080/realms/instasafe-lab"
```

### Single Sign-On Endpoint

```xml
http://44.222.176.144:8080/realms/instasafe-lab/protocol/saml
```

### Metadata Purpose

The metadata XML contains:

- Identity Provider Entity ID
- SAML Single Sign-On endpoints
- Single Logout endpoints
- Signing certificate
- Supported NameID formats

This metadata would normally be provided to a Service Provider during SSO onboarding.

---

# User and Group Configuration

## User

| Attribute | Value |
|------------|------------|
| Username | testuser |
| Email | testuser@instasafe.local |
| Password | TestUser@123 |

## Group

```text
support-team
```

User `testuser` was added to the `support-team` group.

---

# SAML Authentication Flow

A lightweight Flask-based Service Provider simulation was deployed on port 9090.

### Flow

1. User accesses:

```text
http://44.222.176.144:9090/saml/login
```

2. Service Provider redirects the user to Keycloak.

3. User authenticates using:

```text
Username: testuser
Password: TestUser@123
```

4. Keycloak generates a SAML Assertion.

5. The assertion is returned to:

```text
http://44.222.176.144:9090/saml/callback
```

6. User attributes such as email and group membership are included in the SAML response.

---

# Verification

The following items were successfully verified:

- Keycloak admin console accessible
- Realm created successfully
- User account created
- Group created
- User assigned to group
- SAML client configured
- Email attribute mapper configured
- Group attribute mapper configured
- IdP metadata downloaded
- SAML authentication flow tested

---

# Troubleshooting Question

## If a customer reports that SSO users receive an "Attribute Error", what would you check first?

The first item to verify is the SAML Attribute Mapper configuration.

### Menu Path

```text
Clients
→ InstaSafe SP Simulation
→ Client Scopes / Mappers
```

### Items to Verify

1. Email mapper exists.
2. Groups mapper exists.
3. Attribute names match the Service Provider requirements.
4. User profile contains values for mapped attributes.
5. Group membership is correctly assigned.
6. Mapper type is correctly configured.

### Most Common Cause

The most common cause of SAML attribute errors is a missing or incorrectly configured mapper, resulting in the Service Provider receiving an unexpected attribute name or no attribute value at all.

---

# Conclusion

The Keycloak Identity Provider was successfully deployed and configured. Users, groups, SAML clients, and attribute mappers were created successfully. Metadata was generated and exported, and a SAML authentication workflow was validated using a simulated Service Provider.
