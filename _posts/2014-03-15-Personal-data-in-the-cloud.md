---
author: Alex Singleton
layout: post
avatar: alexsingleton
title: Personal Administrative Data in the Cloud
comments: true
categories:
- Cloud
---

##Background

Although such issues are not a new phenomenon, with past cases of [lost USB sticks or laptop computers](http://news.bbc.co.uk/1/hi/uk/7449927.stm), this week again saw controversy surrounding personal data leaking into the public domain. The news was broken by Ben Goldacre on [Twitter](https://twitter.com/bengoldacre/status/440475049880195073) and concerned [Hospital Episode Statistics (HES)](http://www.hscic.gov.uk/hes). Such data are proving to be a real flashpoint (and rightly so) for the public discourse around the use and dissemination of personal information. Ben Goldacre reported that an interface to HES had been developed by the mapping company [Earthware](http://www.earthware.co.uk/), who later clarified that the data displayed were [mock rather than actual](http://www.wired.co.uk/news/archive/2014-03/03/care-data-leaks). Nonetheless, a [HSCIC](http://www.hscic.gov.uk/) investigation is underway. A number of issues were pertinent to this story. The first was how quickly the news broke after Ben's initial Tweets; as a high profile user of Twitter with many followers, this was to some extent inevitable. If you look though the [trail of retweets](https://twitter.com/bengoldacre/status/440475049880195073), responses vary between outrage and to a lesser extent, calls for caution until the facts were known. As it transpires, the HES data appear to have been synthetic (i.e. "made-up") to illustrate how the tool developed might have operated in a more secure environment. However, without this being made clear from the outset, it is easy to see how the opinions that propagated were reached. It is clear that such issues have serious consequences, both in terms of public confidence for the release of open data, but also for those companies involved. Caution is needed.

Earlier in the month, HES data were also [featured in a different story](http://www.theguardian.com/society/2014/mar/03/nhs-england-patient-data-google-servers), where a consulting firm had developed analytical tool to analyse these records from within the cloud. The tools were presented by [PA Consulting Group](https://t.co/romXZYcQoh), and deployed [BigQuery from Google](https://cloud.google.com/products/bigquery/), essentially a cloud based "big data" storage and processing service. This issue has led to a high profile complaint being made to the [Information Commissioner](http://www.cl.cam.ac.uk/~rja14/Papers/ICO-PA-FIPR-complaint.pdf). 

##Considerations

Where personal data are uploaded into a cloud setting, this should stimulate a series of considerations, perhaps most pertinent:

* Are data stored in a cloud setting secure in terms of transfer, storage and access by third parties?
* If data are uploaded into a cloud setting, can you be sure that the data are not being transfered outside of the UK/EU?
* Can assurance be given that data within a cloud setting have been deleted after a given duration?

Such questions are raised by the Data Protection Act, with a useful overview / guidance provided by the [Information Commissioner's Office](http://ico.org.uk/for_organisations/data_protection/the_guide) website. Of particular relevance are sections: [Information security (Principle 7)](http://ico.org.uk/for_organisations/data_protection/the_guide/principle_7) and [Sending personal data outside the European Economic Area (Principle 8)](http://ico.org.uk/for_organisations/data_protection/the_guide/principle_8), which are defined as:

> Appropriate technical and organisational measures shall be taken against unauthorised or unlawful processing of personal data and against accidental loss or destruction of, or damage to, personal data.

> Personal data shall not be transferred to a country or territory outside the EEA unless that country or territory ensures an adequate level of protection for the rights and freedoms of data subjects in relation to the processing of personal data.

The ICO recognise the increasing prevalence of analysis conducted within cloud settings, and have provided a [helpful set of guidance](http://ico.org.uk/for_organisations/data_protection/topic_guides/online/cloud_computing), including very useful disambiguation of those terms commonly used in cloud computing. Furthermore, details can also be found in EU working party article surrounding [data protection within cloud environments](http://ec.europa.eu/justice/data-protection/article-29/documentation/opinion-recommendation/files/2012/wp196_en.pdf). For anyone considering using cloud services, both of these documents are very helpful.
 
 ## Google Cloud Computing
In our project looking at the estimation of CO2 emissions linked to the pupil-school commute, we considered cloud infrastructure as a method by which we could speed up computation, and specifically utilising [Google Compute Engine](https://developers.google.com/compute/docs/faq#selectedcountries). This infrastructure enables the deployment of linux based OS within a remote setting, enabling the benefit of large storage and flexible processing infrastructure. However, a critical constraint for our project was that the specification of countries in the Google Compute Engine could as a minimum only be isolated to the EU. Details can be found [here](https://developers.google.com/compute/docs/faq#selectedcountries), and none of the countries listed were within the UK ([Hamina, Finland](http://www.google.co.uk/about/datacenters/inside/locations/hamina/), 
[St. Ghislain, Belgium](http://www.google.co.uk/about/datacenters/inside/locations/st-ghislain/), 
[Dublin, Ireland](http://www.google.co.uk/about/datacenters/inside/locations/dublin/)). Secondly, as detailed on the [FAQ](https://developers.google.com/compute/docs/faq#selectedcountries) page, Google note:

>"however at this time selection of data center will make no guarantee that your data at rest is kept only in that region"

As such, we took the view that this would result in too greater risk, and indeed, is not disimilar to those issues raised with regards to the PA Consulting Group example given earlier. As such, for our applications we favoured software developed in a secure local environment.

## Future Cloud

Such issues are not just applicable to the use of Google cloud infrastructure, for example, in the comparable [Amazon EC2 services](https://aws.amazon.com/ec2/), regions can be specified as EU, housing data in [Ireland](http://dublin.amazon-jobs.com/how-to-find-us.html), but again, not within the UK. However, a very interesting development from Amazon is the creation of an [AWS GovCloud](https://aws.amazon.com/govcloud-us/) region for the US, designed to address "specific regulatory and compliance requirements" related to the deployment of sensitive data within the cloud. It is easy to envisage how such considerations could be built into a tailored services within a UK context; and this may mitigate some of the issues raised with the use of generic cloud based solutions. To some extent this process is underway through promotion of the Government [G-cloud framework](https://www.gov.uk/how-to-use-cloudstore), offering [approved services](https://www.gov.uk/government/publications/g-cloud-information-assurance-requirements-and-guidance) through the [Cloudstore](http://govstore.service.gov.uk/cloudstore/). As part of the registration process, companies are asked to provide both details about their [security](https://www.gov.uk/government/publications/g-cloud-security-accreditation-application) and [data protection compliance](https://www.gov.uk/government/publications/data-protection-act-checklist-for-g-cloud-suppliers).
 
 








