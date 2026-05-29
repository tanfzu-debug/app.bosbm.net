# BOSBM VPS Rental SOP Documentation Summary

This document outlines a Linux/Forex VPS rental service modeled after MetaQuotes' approach, emphasizing rapid deployment and automated billing.

## Key Service Characteristics

The offering positions itself as a turnkey solution where "customers receive a VPS in 30 seconds with automatic charging and renewal," contrasting with traditional providers like DigitalOcean by offering zero learning curve and integration with EA licenses.

## Technical Infrastructure

The system uses LXD containers across two server locations:
- France secondary server: 194.163.147.81 (4 containers)
- Tokyo primary server: 46.250.248.142 (6 containers planned)

LXD was selected over Proxmox because it's purely software-based, maintained by Canonical, and optimal for Ubuntu hosts.

## Pricing Structure

Five-tier pricing ranges from NT$290 (Starter: 1 core/1GB RAM) to NT$1990 (Enterprise: 8 cores/8GB RAM), with the Pro plan at NT$490 highlighted as the recommended option.

## Financial Projections

The documentation indicates the secondary France server running 4 customers generates NT$1960 monthly revenue against NT$187 operational costs, yielding approximately 10× return on investment.

## Customer Onboarding

The rental process involves selecting a plan through app.bosbm.ai/vps and receiving connection credentials within 30 seconds, with SSH access and monthly automatic billing through BOS purchased accounts.
