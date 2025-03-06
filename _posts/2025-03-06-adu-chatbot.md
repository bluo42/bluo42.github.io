---
title: "Chatbot for ADU Permitting Questions"
classes: wide
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
---

[View Chatbot](https://adu-chatbot.streamlit.app/)

### Introduction

The Accessory Dwelling Unit (ADU) permitting process can be complex, with regulations varying significantly across cities and counties. Navigating zoning codes, setback requirements, parking mandates, and utility connections often requires detailed research and consultation with local authorities. Our ADU Chatbot is designed to simplify this process by leveraging a database of municipal regulations to provide quick, accurate responses to permitting-related questions. 

### Methodology
The chatbot leverages municipal codes, state laws, and planning guidelines organized by city. We combine this regulatory data with our permitting experience and optimize it for LLM comprehension through prompt engineering to deliver accurate, location-specific guidance. The chatbot is currently powered with OpenAI API.

#### Future Enhancements

We plan to incorporate advanced filtering features, allowing users to input specific parameters such as lot size, zoning designation, and existing structures to receive MLS listings that match the criteria. Additionally, integration with GIS data could enable location-based feasibility assessments.

This tool aims to reduce friction in the ADU development process, making it easier for individuals and professionals to access the information they need efficiently.