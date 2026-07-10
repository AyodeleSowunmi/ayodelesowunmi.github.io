---
layout: page
title: Proxmox Linux Template Automation
permalink: /technical-work/proxmox-linux-template-automation/
---

I built a reusable Linux template workflow that automated the repetitive work involved in creating test VMs in my Proxmox homelab. A new VM could receive its Cloud-Init configuration, get an address through DHCP, register itself in DNS, install the NXLog Agent, and automatically enroll with NXLog Platform, where the appropriate configuration was assigned based on its template labels.

My VM provisioning started before I had a dedicated homelab server.

When I started working at NXLog, I needed more resources for running NXLog Platform, Manager, Agents, and multiple test servers. At the time, I was creating VMs locally with VMware on my Windows machine. I would build a base VM, clone it when I needed another machine, and finish the setup manually.

I eventually got a dedicated Mini PC. My manager at the time recommended XCP-ng, so that was the first hypervisor I installed. I tried it, but it never really clicked for me. I had heard about Proxmox before but never had a reason to get into it because I did not have a server. While researching alternatives to XCP-ng, I came across Proxmox again and decided to try it.

That was how my Proxmox homelab started.

## The problem

At first, I continued creating VMs much the same way I had with VMware.

That meant setting the hostname, configuring networking, assigning static IPs, copying my SSH public key from my MacBook, updating hosts files, and installing the NXLog Agent when I needed it.

I was creating test environments for troubleshooting, reproducing support scenarios, KB articles, and technical videos. Repeating the same setup every time I needed a fresh Linux machine got old quickly.

Then I found out about Proxmox templates.

## From templates to automation

While learning how Proxmox templates worked, I discovered Cloud-Init.

Instead of cloning a VM and manually configuring everything afterward, I could use Proxmox and Cloud-Init to provide the hostname, user, SSH public key, network, and DNS settings for each new VM.

I also wanted to stop editing hosts files. Platform, Manager, and multiple Agents needed to communicate with each other, and I wanted to SSH to my VMs from my MacBook without remembering IP addresses.

Once I decided to use proper local DNS, manually assigning static IPs also made less sense. I moved the templates to DHCP, with the router assigning addresses, and built automatic registration into CoreDNS.

The workflow became:

**Proxmox template → Cloud-Init → DHCP → automatic DNS registration → VM ready to use by hostname**

The first time the full workflow worked end to end was with my RHEL 9 template. I cloned the VM, it registered itself in CoreDNS, and after configuring my MacBook to use the homelab DNS server, I could SSH to the new VM by hostname.

That was the point where the whole setup came together for me.

## Expanding the workflow

After RHEL 9, I added RHEL 8 and refined the process across both. I then expanded the template library to include Ubuntu, Rocky Linux, Debian, and RHEL 7.

I already knew from my Linux experience that I could not copy the same networking and DNS configuration across every distribution. I checked how each system managed networking and adjusted the automation where needed.

The next step in my private homelab workflow was automatic NXLog Agent deployment. Most of the VMs I was creating were test systems, so manually installing the Agent and preparing each new system for testing was another repetitive task that made sense to automate.

On first boot, the workflow installed the NXLog Agent and connected it to NXLog Platform. Labels built into each template allowed Platform auto-enroll rules to match the new Agent, enroll it automatically, and assign the appropriate configuration for that operating system.

By that point, I had connected Proxmox templates, Cloud-Init, Linux networking, CoreDNS, first-boot automation, NXLog Agent deployment, and Platform auto-enrollment into one workflow.

That automation is the part I am most proud of. I took tools that solved different parts of the problem and made them work together so most of the repetitive setup happened automatically.

## DNS automation

The DNS registration was the part that made the workflow feel complete.

After getting an address from DHCP, a new VM could discover its IP address and register its hostname with CoreDNS. The server-side automation handled forward and reverse DNS records.

Registration used a dedicated account with limited privileges. The server validated the hostname and IP address, serialized concurrent updates, and used temporary files before replacing the active zone files. DNS registration was also non-fatal, so a temporary failure would not stop the rest of the provisioning process.

## Validation

I did not consider a VM ready just because it booted.

I checked the hostname and IP address, DNS configuration, forward resolution, Cloud-Init completion, and whether DNS registration completed as expected. The DNS update logic was also tested for both A and PTR records.

The strongest published validation is for the RHEL 8 and RHEL 9 clones, with additional template inspection evidence for Ubuntu. The public repository is sanitized, so it does not contain every template or private workflow that existed in the homelab.

A VM could be running in Proxmox and still have broken DNS, failed first-boot tasks, or incorrect configuration. I wanted to check the result, not just whether the VM had started.

## What I learned

I did not start this project with a complete automation plan.

I started with the manual cloning approach I had used in VMware, got tired of repeating the work, found Proxmox templates, learned Cloud-Init, and connected it with DNS and the other setup tasks I was already doing manually.

That is still how I prefer to learn: build something I actually need, find the repetitive or difficult parts, and improve the process as I understand the problem better.

## How the approach evolved

This was the first version of my homelab provisioning and DNS automation, but it was not the last.

I later used what I learned when I redesigned another Proxmox lab for a friend and colleague. I had about five days of physical access to the server and could only work on it after hours. I set up remote access before returning the server and continued the work remotely.

For that environment, I introduced a dedicated homelab network and used dnsmasq for both DHCP and DNS.

My current homelab has evolved again, with another dedicated homelab network and Pi-hole for DNS and network-wide ad blocking.

The tools changed, but the approach started here: find the repetitive work, automate it, use what I learn, and improve the next version.

## Scope

This was a homelab project, not a universal or enterprise image-building platform.

The public repository is a sanitized version of the implementation. Private infrastructure values, secrets, and the vendor-specific NXLog bootstrap are intentionally excluded.

## Repository

The public repository contains the sanitized DNS automation, Cloud-Init and Proxmox configuration examples, validation tests, and supporting documentation.

[Proxmox Golden Image Automation](https://github.com/AyodeleSowunmi/proxmox-golden-image-automation)
