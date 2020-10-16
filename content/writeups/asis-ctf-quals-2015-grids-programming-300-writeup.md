---
title: " ASIS CTF Quals 2015 - Grids (programming 300) Writeup"
slug: "asis-ctf-quals-2015-grids-programming-300-writeup"
date: "2015-05-11 21:27:40.868639"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

Hint:

> In each stage send the maximun [sic] size of area that can be covered by given points as a vertex of polygon in 2D.
> 
> nc 217.218.48.84 12433

When we connect to the server, we're given the following instructions:

{language=python}
~~~~~~~~
In each stage send the maximun size of area that can be covered by given points as a vertex of polygon in 2D.
Are you ready for this challenge?
~~~~~~~~

Sending the string "yes", we're told "OK, OK, lets start" and after a pause, we're given our first set of points, formatted as so:

{language=python}
~~~~~~~~
The points:
[[8, 1], [1, 5], [7, 6]]
What's the area?
~~~~~~~~

This turns out to be a straight-forward programming challenge. We need two pieces of background information:

- Given a bunch of 2D points, the polygon with some set of those points as vertices that maximizes area will be the [Convex Hull](http://en.wikipedia.org/wiki/Convex_hull) of the points.

- The area of a polygon with *n* vertices (x[i], y[i]) can be easily calculated as detailed [here](http://alienryderflex.com/polygon_area/) by:

  {language=python}
  ~~~~~~~~
  0.5 * Σ (x[i-1] + x[i]) * (y[i-1] - y[i])   (indices mod n)
~~~~~~~~

My solver implementation was written in Perl, using [Math::ConvexHull](http://search.cpan.org/~smueller/Math-ConvexHull-1.04/lib/Math/ConvexHull.pm) to calculate the relevant subset of points:

"System Message: ERROR/3 (<string>:, line 33)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: perl
   :number-lines:
   :filename: solve.pl

   #!/usr/bin/perl

   use strict;
   use warnings;
   use diagnostics;
   use feature 'say';
   use TcpSocket;
   use Data::Hexify;
   use Math::ConvexHull qw'convex_hull';
   use constant {
        X => 0,
        Y => 1
   };
   $|=1;

   my $host = '217.218.48.84';
   my $port = 12433;

   my $response;
   my $sock = TcpSocket->new($host, $port);
   my $p;

   $sock->tryread();
   $sock->tryread();
   say $sock "yes";

   while(1)
   {
        $response = '';
        my $last = '';
        while($response !~ /\]\]/)
        {
                my $foo;
                my $ret = $sock->sysread($foo, 2000);
                $response .= $foo;
                if($response ne $last)
                {
                        say "intermediate:";
                        say Hexify($response);
                        $last = $response;
                        exit if($response =~ /answer rapidly/);
                }
        }
        say Hexify($response);
        $response =~ s/^[^\[]*\[//;
        $response =~ s/\][^\]]*$//;
        say $response;
        my @points;
        while($response =~ /\[(\d+), (\d+)\]/g)
        {
                push @points, [$1, $2];
        }
        say "Points:";
        say "  $_->[0], $_->[1]" foreach (@points);

        my $convex_hull = convex_hull(\@points);
        say "Convex Hull points:";
        say "  $_->[0], $_->[1]" foreach (@$convex_hull);
        my $area = 0;
        foreach my $i (0 .. $#$convex_hull)
        {
                $area += ($convex_hull->[$i]->[X] + $convex_hull->[$i-1]->[X]) *
                        ($convex_hull->[$i]->[Y] - $convex_hull->[$i-1]->[Y]);
        }
        $area /= 2.0;
        $area = sprintf("%.1f", $area);
        say "Area: $area";

        say $sock $area;

        $response = $sock->tryread(0.01);
        say Hexify($response);
   }

~~~~~~~~

The biggest annoyance and waste of time for me in solving this challenge is that even though the points are given as whole numbers, and nothing hints at it at all, the server would not accept whole numbers as correct. For instance, if the area was 2, the answer of "2" would be treated as incorrect but the answer "2.0" was accepted. This wasted a significant amount of time for me and it was only after I noticed the pattern did I add the sprintf on line 66 and start submitting answers that were accepted.

There may still be a minor bug in my code or in Math::ConvexHull because it would occasionally submit an answer that was rejected, but it was relatively rare. Despite the ~15 minute runtime, the incorrect answers were just a minor inconvenience and restarting the program was faster than hunting down a subtle bug.

After solving 99 puzzles, the program spit out:

{language=python}
~~~~~~~~
0000: 47 72 65 61 74 20 3a 29 20 0a 43 6f 6e 67 72 61  Great :) .Congra
0010: 74 7a 21 20 41 53 49 53 7b 66 33 61 38 33 36 39  tz! ASIS{f3a8369
0020: 66 34 31 39 34 63 35 65 34 34 63 30 33 65 35 66  f4194c5e44c03e5f
0030: 63 65 66 62 38 64 64 66 36 7d 0a                 cefb8ddf6}.
~~~~~~~~

The final flag being: ASIS{f3a8369f4194c5e44c03e5fcefb8ddf6}
