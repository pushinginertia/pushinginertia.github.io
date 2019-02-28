---
layout: post
title: Usability guidelines for entering credit card numbers on a payment page
tags: [Usability and User Experience]
---

As more people with very little computer experience start shopping online, usability on an online retailer's checkout page is becoming even more important. One common problem that impacts the user experience is a very rigid credit card input that does not permit users to enter spaces or hyphens.

Consider the following two ways to enter the same credit card number:

4500123456781234 vs 4500 1234 5678 1234

Credit card numbers, like phone numbers, are cut into smaller readable slices of 4-6 digits because most humans typically can't easily read and comprehend longer numbers. A significant number of your users will try to enter the credit card number the way it appears on the card with spaces between each group of digits, and will get frustrated when they are told they have entered invalid input. In their minds, they have given correct data. Think of older people in their 50s, 60s, or beyond who are starting to shop online. This is the demographic that will give up the easiest. You don't want to lose a sale because a frustrated user abandons your checkout page.

A computer scientist named [Jon Postel](https://en.wikipedia.org/wiki/Jon_Postel) once said, "be conservative in what you do, be liberal in what you accept from others." Taken in this context, write the presentation layer of a tiered application to be flexible and handle obvious mistakes or input variations. Further down the stack in a service or data layer, maintain data and system integrity by being conservative and strict with input data. With a credit card number, it's easy to normalize the input by stripping out spaces and hyphens to then pass into a service layer for further validation and processing. This enhances the user experience, while still maintaining data integrity.

According to the ISO 7813 specification, a credit card number (commonly referred to in the payment card industry as a Primary Account Number, or PAN) can be anywhere from 8-19 digits in length, although 15 is the most common for American Express and 16 is the most common for Visa and MasterCard in North America. Other parts of the world might vary.

The best validation strategy is a two step process: strip out whitespace and hyphens, then verify that the result is a string containing 8-19 digits.

1. Use the following regex to strip the input: `/[ -]//g`
2. Use the following regex to validate the result: `/^[0-9]{8,19}$/`

Every system has its own architecture and it's difficult to provide a universal recommendation on where to apply this change. In a framework with a backing model object, consider leaving the PAN in the model object as it was entered by the user, then changing code in two places:

1. Change the business logic that performs input validation as described above.
2. Change the business logic that passes the PAN to the payment processor (or a service layer that talks to the payment processor) to strip whitespace and hyphens from the input.

The reason for this recommendation is that if the page is rendered on the screen a second time, changing the value in the model object will change the value that the user sees on the screen. This might confuse the user. If you're dealing with a system that is difficult or impossible to change, a bit of Javascript in the rendered html will work to strip the input appropriately. It's not ideal, but will make the input more forgiving for the user.

Finally, the `<input>` tag should be set to have a maximum length of 22 so that the user can enter a 19 digit number in 4 slices.

This concept can be abstracted and applied to other inputs as well. If you're accepting international orders, keep in mind that postal code formats vary in different countries. In the U.S., a zip code can be 5 or 9 digits. In Canada, a postal code is commonly written in three different ways: A9A9A9, A9A 9A9, and A9A-9A9. Australia's postcode uses a 4-digit number. If you're doing address verification, check with your processor on which format they prefer when there is ambiguity.
