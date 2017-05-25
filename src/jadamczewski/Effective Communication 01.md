## Effective Communication: Code Review

Key ideas:

1. Communication isn't just about delivering information; at its best, communication is a two-way process.
2. Respect is an important part of communication. If the person you're communicating with feels that you value their time, they are more likely to give your their attention.

Concretely, these ideas can be seen in action in a team that does code reviews: there is a request for review, some description of the change, and some back and forth discussion about the purpose, details, and merit of that change.

### Seeking review

When seeking code review, value the time of the reviewer - good code review requires a lot of attention and consideration, so anything that distracts the reviewer from the review will subtract from what they have to put into the review.

The most important principle I have when seeking code review is this:
_Make your change as easy as possible to review._

I say this because I know what gets in the way of me reviewing others' code. Do I have to do more than click a single link in an email? Can I just walk to someone else's desk where the change is already on the screen? Great! If there's more than that - if I have to dig the change out of the perforce depot, or copy and paste from an email (or review code sent in an email - ugh!), or I have to hunt someone down to review their code, I'm going to be more reluctant to do the work, and less inclined to give it my full attention.

### Target your request

Depending on the team, it may not always be clear who is the best person to review the change. If that is the case, it is better to not broadcast the request to every programmer in the company.

Ideally, ask specific people who have ownership or familiarity with the code to review your changes. If you don’t know who those people might be, ask someone else who might know. It shouldn't take too long to identify suitable reviewers. 

### Be a reviewer

The other side of showing respect to for others time is to resond when they seek help. Take the time to review other people’s changes. It is frustrating to seek review and hear crickets. If you are not comfortable with being the [only] reviewer of a change, say so _and suggest someone else who should review the change_.

### Documentation is communication

Documentation is an important part of the review process. Documentation of the change, the purpose of the change, and the effect of the change make it easier to review, and easier to maintain.

Before asking for review, add comments (in the code and/or the review system) to make it easy for a prospective reviewer to understand why the solution is implemented the way that it is, the intended behavior. Writing tests can also help communicate what you are expecting of the code, and demonstrate that the code works for at least those tests.


