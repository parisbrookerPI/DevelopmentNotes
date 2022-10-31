What happens if an event fails to publish? Doesn't really matter if there's await or not before calling the event publisher (i.e. aditional latency), data integrity is the main issue.

(Lecture 347)
Solution:

2-Stage event publishing + database transactions (though not implemented in this course).