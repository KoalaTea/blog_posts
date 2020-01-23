# Testing
This post is majorly distilled from Ian Coopers talk (https://www.youtube.com/watch?v=EZ05e7EMOLM), Googles Testing on the Toilet Series (https://testing.googleblog.com/search/label/TotT), Martin Fowlers testing related blog posts (https://martinfowler.com/articles/practical-test-pyramid.html), and Rob Moores posts (https://robdmoore.id.au/blog/2015/01/26/review-of-ian-cooper-tdd-where-did-it-all-go-wrong)

## Types
## Unit Testing
## Integration Testing
## Functional Testing

## Testing Pyramid

## Methodologies
## TDD
## Red Green Refactor
Ian suggests that the original TDD Flow outlined by Kent Beck has been lost in translation by most people. This is summed up nicely by Steve Fenton in his summary of Ian’s talk (highlight mine):

    Red. Green. Refactor. We have all heard this. I certainly had. But I didn’t really get it. I thought it meant… “write a test, make sure it fails. Write some code to pass the test. Tidy up a bit”. Not a million miles away from the truth, but certainly not the complete picture. Let’s run it again.

    Red. You write a test that represents the behaviour that is needed from the system. You make it compile, but ensure the test fails. You now have a requirement for the program.

    Green. You write minimal code to make the test green. This is sometimes interpreted as “return a hard-coded value” - but this is simplistic. What it really means is write code with no design, no patterns, no structure. We do it the naughty way. We just chuck lines into a method; lines that shouldn’t be in the method or maybe even in the class. Yes - we should avoid adding more implementation than the test forces, but the real trick is to do it sinfully.

    Refactor. This is the only time you should add design. This is when you might extract a method, add elements of a design pattern, create additional classes or whatever needs to be done to pay penance to the sinful way you achieved green.

    When you do this right, you end up with several classes that are all tested by a single test-class. This is how things should be. The tests document the requirements of the system with minimal knowledge of the implementation. The implementation could be One Massive Function or it could be a bunch of classes.

Ian points out that you cannot refactor if you have implementation details in your tests because by definition, refactoring is where you change implementation details and not the public interface or the tests.
## DBB

## Good tests

## Splitting up tests that rely on other systems

## Testing correctly

### Further links
#### Blogs
- https://medium.com/javascript-scene/mocking-is-a-code-smell-944a70c90a6a
- https://martinfowler.com/articles/practical-test-pyramid.html
- https://dave.cheney.net/2016/08/20/solid-go-design
- https://dave.cheney.net/2016/04/11/the-value-of-tdd
- http://natpryce.com/articles/000813.html
- https://robdmoore.id.au/blog/2015/01/26/review-of-ian-cooper-tdd-where-did-it-all-go-wrong
#### Videos
- https://www.youtube.com/watch?v=Eu35xM76kKY
- https://www.youtube.com/watch?v=EZ05e7EMOLM
    - https://vimeo.com/68375232
#### Books
- https://smile.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530?sa-no-redirect=1