# apollo-questions

Hey, so I've got this design where we have a carousel of all practitioners you've previously visited, and a list of companies / clinics that you've linked to your central account.

![booking tab](BookingTab.png)

Initially my thoughts when looking at this design was "great, this a a classic use case for GraphQL, I can combine both queries into one". The problem however, is that both practitioners and companies are paginated. So if I wanted to fetch more practitioners, I'd have to refetch all companies data as well, and vice versa when calling the `fetchMore` function.

Because of this, I ended up making two separate components, one for the Practitioners carousel, and one for the Company list - each of which are responsible for fetching their own data as separate queries, and each component can handle pagination individually without the other component rerendering or refetching.

The problem now, is that we have 2 separate loading states in sibling components. Our designer would like the skeleton in the main page to only hydrate with the actual components when **all** data has been fetched, instead of hydrating each section as the data becomes available, which means I now either have to move both queries into the main parent page and pass the data down, or manage the laoding state of both components in the state of the parent page.

## My question is this:

Is there a way to do partial pagination in a single query in Apollo Client? So keep the Practitioners and Companies in the same query so that everything loads initially in the same request but when fetching more practitioners, only fetch the new practitioners, and not refetch the companies as well, and vice versa?

Is it a viable approach to have the parent component fetch the initial data, and then have lazy queries in the sibling components that can be used to handle the pagination separately?

Is there another Apollo client approach / pattern that solves this scenario?

This is how the individual queries look:

```graphql
query MyPractitioners($amount: Int!, $cursor: String) {
  myStaffMembers(first: $amount, after: $cursor) {
    edges {
      node {
        id
        avatarUrl
        prefix
        firstName
        lastName
        suffix
        isAvailableToday
        discipline
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}

query MyCompanies($cursor: String, $amount: Int!) {
  myCompanies(after: $cursor, first: $amount) {
    edges {
      node {
        companyGuid
        name
        streetAddress
        logoUrl
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```
