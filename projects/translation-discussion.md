
## Translation Discussion

Chinese and Japanese Translations  

Keeping in mind the ease-of-use of in-browser Google Translation, does it makes sense to put significant effort into a translation project?

 And will the quality of the AI translations exceed Google?

From Janko:
Is this really a problem nowadays when browsers can translate pages for you?

Will's comment: Google Translate is fine for one‑off reads, but quality varies by browser/time and we can't control it. There's no easy way to ensure consistency or QA. A pipeline combining machine translation with human review (largely from Chinese‑speaking contributors through a customized `Weblate` server; demo coming soon) would yield consistent Chinese library docs. This effort may also improve the quality of the English documentation.

From Mungo Gill:
Depending on the source and target language, automated translation quality varies between almost acceptable and plain wrong.

From Alan de Frietas:
Coding is much harder than technical English. In the rare case where language is the barrier, the person already has lots of alternatives for automated translations. Most browsers already come with it, and many even annoyingly enable it by default. Our time is always better spent on better English documentation, so the person actually has the content to translate.

Will's comment: Many (Chinese) developers find technical English slow or hard to read. Translated docs can accelerate understanding.

From Alfredo Correa:
technical language is different from everyday language. for technical stuff you usually loose more and risk more from a translation that what you gain. yes, language is a barrier, but bad translation of technical material is a barrier too, and perhaps a worst one.
I don’t speak Chinese or Japanese, I don’t know if they are good technical languages. It happens that English is a decent technical language, because it is simple. The language of math, science and programming. It could have been (and was at a time) French, or German, but it happens that English is the technical language for the time being.
Writing documentation in “simple english” might be more valuable than translations, even to popular languages, like Chinese or Spanish. see for example https://simple.m.wikipedia.org/wiki/Simple_English_Wikipedia

## Implementions

### Option 1: Vertical Integration

Enable internationalization options in Django. Move all text strings into translation files. Configure the load balancers and SSL certificates of the cluster to accept cn.boost.org and jp.boost.org domains in addition to the current domains. Alternatively, there can be query parameters in each URL string &lang=cn and then continue with www.boost.org, however the full-fledged domains might be cleaner.  

On the backend, copy/clone/mirror the AWS S3 bucket that contains the boost library docs.  
Likewise, copy/clone/mirror the AWS S3 bucket that contains the website-v2-docs content.

Give an AI write-access to the foreign language S3 buckets. It modifies/translates the content.

Advantages: The most integrated solution.  

Disadvantages: This change affects all Django developers. All the time. It's no longer possible to just write a text string into a random Django website-v2 file. Everything must be translated. If you're developing a new feature, it slows down the velocity of development of the website, on a day to day basis. This is inconvenient and somewhat expensive in terms of the current team's efforts, although not impossible...  as they must constantly be modifying the code to support the foreign language translations when developing any webpage.  

### Option 2: Vertical Integration - Reduced version

Similar to Option 1. Limit translations to the S3 bucket content. Not all pages on the site.   
- translate boost library docs  
- translate website-v2-docs guides  
- leave Django mostly unmodified

Advantages: At least the "docs" have been translated. It's easier to maintain than Option 1. It doesn't affect the development team as much.  

Disadvantages: Chinese or Japanese visitors would need to navigate many webpages that are still in English.  

Will's comment: I agree full UI i18n would slow `website‑v2`. Since we're in the process of deploying and testing a separate docs translation server (`Weblate`) and `website‑v3` is coming, I suggest focusing now on translating library docs and guides. For `website‑v3`, it's worth adding proper i18n (e.g., URL prefix (`/zh/`), language dropdown, S3 language‑prefix folders, DB translation fields with English fallback etc).

### Option 3: Separate Mirror. Most Basic Version.

- Host the translated site on a very basic, static HTML server. Just one nginx web server. It can be backed up on a daily basis so it's recoverable. That is not a problem.  The cloud is reliable.
- Translate boost library documentation and website-v2-docs guides.  
- The technology could use simple PHP pages, or HTML, to navigate through the docs. No Django. Perhaps no S3 bucket. Copy all the files locally. The source of the copy is the S3 bucket. Have an AI translate the docs.
- It should be acceptable in this scenario that the functionality and the navigation of the website are severely reduced. A lot of features are missing. Perhaps the main "Releases" and "Libraries" pages are gone. What's being provided is a mirror of the documentation, translated.  

If the goal is to add more and more features to Option 3 over time, and eventually achieve feature-parity, then Option 3 doesn't make sense. It would be reinventing-the-wheel. The only way this Option is reasonable is if it's acceptable to forego many features, maybe the version-dropdown is gone, maybe the Releases page is gone. Just show some basic documentation.

Advantages: Simpler to build. Doesn't interfere with production at all. All pages (that are shown) will be translated. No English pages.      

Disadvantages: Many features are missing, in this "basic" scenario.

### Option 4: Separate Mirror. Advanced version.

Fork the website-v2 github repository, to a Chinese or Japanese version. Host a fully featured stack, with all components (separate GKE K8S cluster, database, redis memorystore, S3 buckets).

This doubles the infrastructure efforts. It's very expensive in terms of manpower, knowledge, hosting, finances, and everything. Hire a Chinese or Japanese team to maintain this.    

A question - how would day-to-day code updates be migrated from the website-v2 to the website-v2-jp repo? Maintaining the git forks, just this part, would seem to be problematic and difficult.    

Advantages: Doesn't interfere with the existing website-v2 development.  
Disadvantages: Doubles all costs. Must hire many more people.   

