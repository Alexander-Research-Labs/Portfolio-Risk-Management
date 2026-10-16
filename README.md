# Portfolio Risk Management 
Disclaimer: Informational and educational purposes only. Nothing in this repository, including its VaR/CVaR output, Kelly sizing suggestions, or any other result, is a recommendation, offer, or solicitation to buy, sell, or hold any security, and none of it should be construed as investment, financial, legal, or tax advice. This is a personal project, not a product of a registered investment adviser, broker-dealer, or fiduciary. The code and its output are provided "AS IS," without warranty of any kind: simulated results depend entirely on model assumptions and calibration choices, may contain errors, and are not guaranteed to reflect real-world outcomes, past or future. Anyone using this code is solely responsible for their own investment decisions and should consult a licensed financial professional before acting on anything produced by it.


## Important: Please Read the thesis.

## Purpose

The purpose of this repository is to showcase my design of stress-testing my portfolio and to determine the most optimal quantity of an equity to buy with AI-assisted code.

## Known Flaws/Limitations
No backtesting for CVaR since there are too few data points.
Variance should mathematically never go negative, but the math used to simulate it accidentally produces a negative number. Instead of a more complex fix, the tool just changes it to zero whenever that happens. Not a unique issue to this tool and not something that needs fixing.
VaR/CVaR describe a confidence level, losses beyond the reported number remain possible by construction.

## Other

This research project was made after my research project on the repository "Geometric-Brownian-Motion-for-VaR-and-Equal-Risk-Contribution," so excuse the "out of order" feel and similar study topic and method it might have. I may or may not release that project.

Originally built for my private portfolio tools.
