# Smarter AI Document Processing with Enrichment Fingerprinting

Reducto’s vision models provide the best document processing accuracy on the market today, but we’re always pushing for better performance. Accuracy is key: we can [outperform vision language models](https://reducto.ai/blog/sota-table-parsing) (VLMs) such as 4o without the [adverse side effects of hallucinations](https://reducto.ai/blog/lvm-ocr-accuracy-mistral-gemini). But VLMs have their strengths, like interpreting unclear symbols or handwriting, both commonplace in the millions of documents we process daily. 

That's why we correct and supplement Reducto outputs with VLMs.

&lt;graphic of Reducto&gt; 

## Introducing Enrichment Fingerprinting

Enrichment Fingerprinting is a feature designed to match and align structured data extracted by Reducto with AI-generated results from large vision models. Since AI models often interpret documents differently, we establish mappings between our parsing blocks and VLM outputs.

### How does it work?  

The system breaks down text into small word sequences ([n-grams](https://en.wikipedia.org/wiki/N-gram)) and identifies unique patterns within both Reducto’s structured output and the VLM-generated text. Think of an n-gram as a phrase or sequence of words, such as “Invoice total” or “Total amount due”. 

If we can identify that these two mean the same thing and find a unique, one-to-one match, we can link the VLM’s understanding of the content to Reducto’s structured data. 

With these n-grams, we can calculate a probability matching: for example, using simple Bayesian conditional probability with word counts, or discounting with [Katz’ backoff ](https://en.wikipedia.org/wiki/Katz%27s_back-off_model)by setting aside some probability mass (or some other smoothing method). In practice, we use a variety of methods to reduce the overall [perplexity](https://en.wikipedia.org/wiki/Perplexity) (per word) of how our n-gram model predicts matching samples.  

Sometimes, there’s no exact or even good match between the two outputs. We have multiple strategies to resolve these differences:

- Bag-of-Words Similarity: Instead of looking for exact n-length phrases, we can compare the overall similarity score of a text grouping. 

- Merging Blocks: If one line from the model corresponds to multiple smaller pieces in our output, we can combine them into a single unit. 

- Verifying Block Adjacency: Before merging, the system checks if the blocks are logically related. We utilize our ability to identify boundaries in documents to accurately do this, without merging unrelated information by mistake. 

- Default Text Inclusion: If a piece of text can’t be confidently matched with less than 90% accuracy, we still include it. This way, we avoid missing critical information loss that is common with utilizing VLMs alone. 

## Case study 1: Medical document 

source: [Kaggle](https://www.kaggle.com/datasets/mehaksingal/illegible-medical-prescription-images-dataset)

[Sample without EF](https://app.reducto.ai/share/4cb71f18-602e-42af-85c4-5e406619c9eb)

[Sample with EF](https://app.reducto.ai/share/1c64bcb6-bd8d-4b3e-b0b5-629752da1797)

Here’s a simple example: a handwritten medical receipt that includes symbols (such as the logo of the company) that are accurately captured with the VLM. We combine this result with the bounding of the logo-ed area of the form captured by our vision models to get a complete, accurate understanding of the company header. 

While the above is merely an example document, [HIPAA compliance is supported for Scale and Enterprise tiers. ](https://docs.reducto.ai/security/policies)

## Case study 2: IRS Retirement Plan Form

Source: [Unstract ](https://unstract.com/blog/guide-to-extracting-data-from-handwritten-pdf-with-ocr/)

[Sample without EF](https://app.reducto.ai/share/663ec30d-7a5d-4b4f-84d7-75abc5405924)

[Sample with EF](https://app.reducto.ai/share/0c32a21e-daa9-408a-916d-711aff2d029d)

In this document, we run into a common scenario where a combination of multiple informational hierarchies (eg., nested inputs, multiple sub-sections) make for a difficult interpretation for standard VLMs to typically handle independently, much less accurately bound. 

With Enrichment Fingerprinting, we can accurately capture and merge the blocks involving the Financial Information in part (III), and correctly interpret the boxes in sections 5a(1) - 5b(1). Without it, we consider these as symbols checked. 

Note that these are being captured without any custom prompting. 

## Challenges and handling edge cases 

Most documents don’t follow a simple structure, so the system needs to handle complex scenarios such as: 

- Overlapping or Nested Information: If one section of the document contains multiple matching blocks, we find the best way to split or combine them.

- Duplicate or non-unique n-grams: If the same n-grams appear in different parts of the document, we ensure they get linked correctly based on the context surrounding the individual text groupings. 

- Content Spanning Across Sections: If a sentence starts in one section and finishes in another, we merge them properly.

## The future of Enrichment Fingerprinting

Incorporating vision language models with our more traditional computer vision approach allows us to have the best of both worlds: we’re able to accurately understand complex layouts and bound information sources in documents, while also incorporating the scenarios that these AI models excel at. Eventually, we believe this can lead to accuracy rivaling the best human classifiers while avoiding the pitfalls of hallucinations or data loss from utilizing VLMs alone. 

This is just an early look at what’s to come. For more, you can try out the feature [here](https://app.reducto.ai/) or reach out to us for a demo. 
