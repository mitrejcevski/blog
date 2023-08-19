---
title: "Android CI/CD using GitHub Actions - Complete Setup"
date: 2023-08-14T00:26:55+02:00
featured_image: "/images/ci-cd/CD.png"
tags: ["Android", "Testing", "CI/CD"]
draft: true
---
# Introduction

### CI/CD stands for Continuous Integration/Continuous Delivery

One of the things computers are best at is performing repetitive operations over and over again. They are never getting bored or tired, rather, they are happy to do those tasks for us.

The idea behind CI/CD is to establish a workflow that allows us to deliver the work we do to the customers quickly, reliably, and frictionlessly.

It is a system that enables us to trigger a whole set of operations that are working in a sequence or parallel by a simple push to a remote repository. These operations can be anything from running custom scripts that we can write ourselves, all the way to writing release notes based on the changes done from release to release. For most of the projects, most of the time, the CI/CD does at least the following tasks:

Pull the latest code changes

Build it

Run checks (like lint and tests)

Deploy

Doing that sequence of tasks every time manually is super painful, time-consuming, mistake-prone, and gets very boring over time. Establishing a CI/CD system will enable us to run the sequence as many times as we want effortlessly. So it is a great return on investment.

In this series of articles, we will get through the steps to set up CI/CD for deploying an Android App to Google Play Store, and we will look into many interesting things along the way. Enjoy!

