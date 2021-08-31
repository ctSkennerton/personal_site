---
title: 'Open Source Tax Software'
date: Mon, 30 Aug 2021 18:45:28 -0700
---

Filing taxes in America sucks. Your options are to do it by hand, pay someone like Intuit, or
if you are below a certain income threshold get some tax software for free. The kicker is that
free tax software is from Intuit who will try very hard to make sure that you either don't find
it in the first place, or try to get you to pay for it and upsell you on something that should
be free. Check out this [excellent article](https://www.propublica.org/article/inside-turbotax-20-year-fight-to-stop-americans-from-filing-their-taxes-for-free) from ProPublica that outlines Intuit's tactics.

I've tried to do my taxes by hand and made mistakes that mean I'd have to amend my return. 
This tax year I was pleasantly surprised to find https://excel1040.com, which offers
a fairly comprehensive list of federal tax forms. The beauty of excel1040 is that the forms
themselves have been recreated in excel, one form per tab. It's incredibly powerful to see the
forms get filled out automatically as you put in input values from your W-2 or 1099 forms.
And since it's excel you can freely overwrite values if you think they are wrong or add in
new logic if a form isn't implemented yet. The author even says that you can print out the
forms from excel and they'll be accepted, though I didn't try that and instead copied the
values from the spreadsheet onto the pdf forms from the IRS.

Though I'm sure there are unsupported parts of the
tax code in excel1040 it does have a wealth of different forms and the author appears to have
been slowly evolving the complexity of the task over many years.

The only downside with excel1040 is that it's tricky to understand how all of the formulas
are implemented as there is some complex logic that has to be squeezed into a single line
formula. Here are a couple of examples of what I mean:

```
=IF(P74<>"",P74,IF(NOT(F6251_PIII),"",IF(AND(SchDTW_Used,NOT(CGTW)),'Sch. D WS'!P87,IF(CGTW,'Line 16 (QDCG Tax)'!J34,0))))
=IF(AM44<>"",ROUND(AM44,0),IF(OR(AM25,AM29),"",IF(AL6<>"",IRA_CONV_YOU,IF(AL7<>"",IRA_CONV_SPOUSE))))
```

After I found excel1040, I also discovered https://ustaxes.org which is aiming to be a replacement
for paid tax software. It's a website and downloadable application that lets you add in your
income and it does the math and fills in the IRS pdf forms automatically. It is written in 
typescript and felt more natural to me (as a programmer) to understand the logic. I've already
contributed a couple of additions and plan to keep contributing. It doesn't have 
as many of the forms implemented as excel1040 but I see it having at least one advantage:
Because it is written in typescript as a custom application, we can take advantage of software
tools like automatic testing, version control, and better structure to the formulas and logic.

I hope that efforts like excel1040 and ustaxes.org get more popular so that Americans have
additional, free choices when filing their tax.
