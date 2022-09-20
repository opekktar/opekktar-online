---
title: "Countdown_update"
date: 2022-09-05T10:52:09-07:00
draft: false
tags: [Life, Hope, Countdown]
catagory: Blog
---
<h1 ><p id="blink"> UPDATE UPDATE UPDATE </p></H1>
    <script type="text/javascript">
        var blink = document.getElementById('blink');
        setInterval(function() {
            blink.style.opacity = (blink.style.opacity == 0 ? 1 : 0);
        }, 750);
    </script>

# 9 Years is an odd number; Lets go for a full decade

***Let's make this like college!!!*** Let us pretend the first 8.2 years was 2 weeks before the exams and now it's the night before and time to cram for the exam!!! Ironically I was never as stupid in school as in life. I'm moving the goal post because I do not want to off myself for lack of motivation, because I lack the mo...tif..vat.......ion to do so; I'm moving the goal post at the 11th hour:<!-- Display the countdown timer in an element -->
<p id="second"></p>

<script>
// Set the date we're counting down to
var countDownDate = new Date("Oct 17, 2023 03:37:25").getTime();

// Update the count down every 1 second
var x = setInterval(function() {

  // Get today's date and time
  var now = new Date().getTime();

  // Find the distance between now and the count down date
  var distance = countDownDate - now;

  // Time calculations for days, hours, minutes and seconds
  var days = Math.floor(distance / (1000 * 60 * 60 * 24));
  var hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  var minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
  var seconds = Math.floor((distance % (1000 * 60)) / 1000);

  // Display the result in the element with id="demo"
  document.getElementById("second").innerHTML = days + "d " + hours + "h "
  + minutes + "m " + seconds + "s ";

  // If the count down is finished, write some text
  if (distance < 0) {
    clearInterval(x);
    document.getElementById("second").innerHTML = "DECADE EXPIRED";
  }
}, 1000);
</script>

There now I have something to blog about, yay[^3]

[^3]: An expresion of joy or `Yet another yogurt. Pacman wrapper and AUR helper written in go.` Though I use the latter every day; I meant the former in this case. Yep! I'm a nerd!
