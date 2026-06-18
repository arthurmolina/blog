---
title: "Better ways to identify individual records in Ruby on Rails"
lang: en
last_modified_at: 2023-06-04T20:00:00-03:00
categories:
  - articles
tags:
  - nonsense
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
toc: true
---

When we want to access a specific model record, the easiest way is to use the unique identifier as an access key. By default in SQL databases we have a sequential integer primary key. But this is not the only possible option. We also have the possibility of using UUID, Slug and Hash ID or Token. Each method offers distinct advantages and considerations, impacting factors like security, readability, and database performance. Drawing insights from real-world implementations and industry best practices, we explore the nuances of each method and their implications for modern applications. Let's compare these approaches and explore why platforms like YouTube opt for Hash IDs.

# Sequence IDs: Embracing Predictability and Efficiency

This is the easiest option to implement as it is the one available in most cases. When we create a new model or generate a scaffold, the Sequence ID is already the established standard. Sequence IDs maintain a predictable numerical sequence, facilitating database operations like range queries and indexing. 

## Advantages

1. **Sequentiality**: Sequence IDs maintain a predictable numerical sequence, facilitating database operations like range queries and indexing.
2. **Simplicity**: Sequence IDs align with conventional database practices, requiring minimal configuration or external dependencies.
3. **Efficiency**: Sequential IDs promote efficient database indexing and storage, optimizing performance in certain use cases.

## Disadvantages

1. **Predictability**: Sequential IDs expose database sequence information, potentially enabling attackers to infer patterns or exploit vulnerabilities.
2. **Scalability**: In distributed environments or high-throughput applications, managing sequential IDs across multiple nodes can pose scalability challenges.
3. **Readability**: Numeric IDs lack the human-readable qualities of hashed strings, potentially impacting user experience and URL aesthetics.

## Potentialities

1. **Performance Optimization**: Sequence IDs excel in scenarios where sequential access or range queries are prevalent, leveraging database indexing for enhanced performance.
2. **Compatibility**: Sequence IDs seamlessly integrate with existing database schemas and workflows, minimizing migration efforts and compatibility concerns.

## How to do this

You don't need to do anything beyond the basics. This is already the standard.


# UUIDs (Universally Unique Identifiers): Ensuring Global Uniqueness and Interoperability

Universally Unique Identifiers (UUIDs) provide unparalleled guarantees of global uniqueness across distributed systems and time. This decentralized approach to identifier generation alleviates concerns about collision risks, making UUIDs ideal for large-scale deployments and decentralized architectures. However, the very long IDs and the lack of connection with more human information end up making their use difficult. It is a good option for those who have distributed databases or want to merge databases. I wouldn't use it if I didn't have one of these reasons.

## Advantages

1. **Uniqueness**: UUIDs guarantee global uniqueness across distributed systems and time, mitigating collision risks even in large-scale deployments.
2. **Decentralization**: UUIDs can be generated without centralized coordination, making them suitable for decentralized or distributed architectures.
3. **Standardization**: UUIDs adhere to established standards like RFC 4122, ensuring interoperability and compatibility across platforms and frameworks.

## Disadvantages

1. **Length**: UUIDs are longer than sequential or hashed IDs, potentially impacting database storage and network transmission overhead.
2. **Readability**: UUIDs lack human-friendly qualities, often appearing as random strings of characters, which can diminish URL aesthetics and user experience.
3. **Indexing Complexity**: Indexing UUIDs in databases may incur performance penalties due to increased storage requirements and index fragmentation.

## Potentialities

1. **Global Uniqueness**: UUIDs provide unparalleled uniqueness guarantees, making them ideal for scenarios where collision avoidance is paramount.
2. **Decentralized Generation**: UUIDs facilitate distributed system architectures by enabling decentralized ID generation without centralized coordination.
3. **Interoperability**: Standardized UUID formats promote interoperability and compatibility across diverse platforms, frameworks, and technologies.

## How to do this

You've probably already come across several articles explaining how to use UUID in Rails so I'll just put the summary version. So far I've only used UUID in Postgres database, so if you use MySQL I'll be in debt.

```ruby
# /db/migrations/0000000000_enable_pgcrypto_extension.rb
class EnablePgcryptoExtension < ActiveRecord::Migration[6.1]
  def change
    enable_extension 'pgcrypto'
  end
end

# /config/initializers/generators.rb
Rails.application.config.generators do |g|
  g.orm :active_record, primary_key_type: :uuid
end

# /db/migrations/000000_create_users.rb
class CreateUsers < ActiveRecord::Migration[6.1]
  def change
    create_table :users, id: :uuid do |t|
      t.string :name
    end
  end
end
```

# Hash IDs: Balancing Security and Readability

Hash IDs represent a cryptographic approach to generating unique identifiers by transforming numerical values into hashed strings. This technique offers several advantages, including enhanced security, improved readability, and collision avoidance. By obscuring database sequences, Hash IDs mitigate the risk of enumeration attacks and unauthorized access. Platforms like YouTube leverage Hash IDs to create user-friendly URLs, enhancing usability and brand recognition. However, Hash IDs also pose challenges, such as non-sequentiality and dependency on hashing algorithms. Despite these considerations, Hash IDs remain a popular choice for applications prioritizing security and user experience.

## Advantages

1. **Security**: Hash IDs obscure the underlying database sequence, enhancing privacy and deterring malicious activities such as data scraping and enumeration attacks. By transforming numerical IDs into hashed strings, sensitive information remains hidden from unauthorized access.
2. **Readability**: Hash IDs can be tailored to generate human-readable strings, enhancing user experience and URL aesthetics. Platforms like YouTube leverage Hash IDs to create user-friendly URLs, improving usability and brand recognition.
3. **Collision Avoidance**: Hash algorithms strive to minimize the likelihood of collisions, ensuring the uniqueness of generated IDs within practical constraints. While collisions can still occur, modern hash algorithms maintain a high level of collision resistance, reducing the risk of duplicate IDs.
4. **Customization**: Hash IDs offer flexibility in tailoring encoding schemes and incorporating application-specific logic. Platforms like YouTube can customize Hash IDs to incorporate additional metadata or encoding schemes, optimizing resource management and user experience.


## Disadvantages

1. **Non-Sequentiality**: Hash IDs are non-sequential, which may complicate certain database operations like range queries or sorting. While platforms like YouTube prioritize security and user experience over sequentiality, developers must carefully consider the impact of non-sequential IDs on database performance and query optimization.
2. **Dependency on Algorithms**: The effectiveness of Hash IDs hinges on the cryptographic robustness of underlying hashing algorithms. Weak hash algorithms or improper configuration may compromise security and expose vulnerabilities to attacks like hash collisions or brute force decryption.
3. **Limited Encoding Space**: Depending on configuration, Hash IDs may have a finite encoding space, potentially leading to collisions in high-throughput environments. Balancing encoding space and collision resistance is essential to ensure the scalability and reliability of Hash IDs in large-scale deployments.

## Potentialities

1. **Enhanced Security**: With proper configuration, Hash IDs can bolster data security and mitigate risks associated with exposing numerical IDs. By obscuring database sequences and deterring enumeration attacks, Hash IDs contribute to overall system security and integrity.
2. **Improved User Experience**: Hash IDs enable the creation of user-friendly, readable URLs, enhancing usability and brand recognition. Platforms like YouTube leverage Hash IDs to optimize user experience and facilitate content discovery through intuitive, descriptive URLs.
3. **Scalability and Performance**: Hash IDs promote efficient indexing and retrieval in high-throughput environments, ensuring optimal performance and scalability. By balancing collision resistance and encoding space, Hash IDs facilitate seamless resource management and query optimization in large-scale deployments.
4. **Customization and Optimization**: Hash IDs offer flexibility, enabling tailored encoding schemes and metadata incorporation. Platforms can optimize Hash IDs to meet specific application requirements, enhancing resource management, and user engagement.

## YouTube's Hash ID Implementation: A Case Study

YouTube, one of the world's largest video-sharing platforms, utilizes Hash IDs extensively for various reasons:

1. **Security**: Hash IDs obscure the underlying database sequence, enhancing privacy and deterring malicious activities such as data scraping or enumeration attacks.
2. **Readability**: Hash IDs allow YouTube to generate user-friendly video URLs, enhancing usability and brand recognition.
3. **Consistency**: Hash IDs provide a consistent format for video identifiers, simplifying resource management and URL generation across the platform.
4. **Scalability**: With millions of videos uploaded daily, Hash IDs facilitate efficient indexing and retrieval, ensuring optimal performance and scalability.
5. **Customization**: YouTube can customize Hash IDs to incorporate additional metadata or encoding schemes, further optimizing resource management and user experience.

By leveraging Hash IDs, YouTube maintains a balance between security, scalability, and user experience, reinforcing its position as a leading video-sharing platform in the digital landscape.

## How to do this

We have the perfect hashing algorithm for this work: Hashids. There are several different Gems that do the same work as the code above, but I have a philosophy that says that if a file solves the problem there is no need to add another external library resulting in more complexity and problems when updating. There is no need to [have a gem and have to deal with the hassles of maintaining it](/blog/minimizing-gems-libraries-in-ruby-on-rails/). But the same way that I did for Gravatar, I opened [hashid-rails gem](https://github.com/jcypret/hashid-rails) and took what is important:

```ruby
# /app/models/concerns/hashable.rb
# frozen_string_literal: true

# Hashids - Change the numeric identifications to hash identifiers
# Based on a simplification of 'hashid-rails gem' but avoiding yet another gem that could be a concern file.
module Hashable
  extend ActiveSupport::Concern

  def hashid
    self.class.encode_id(id)
  end

  def to_param
    hashid
  end

  # Methods for the Model Class
  module ClassMethods
    def encode_id(ids)
      if ids.is_a?(Array)
        ids.map { |id| hashid_encode(id) }
      else
        hashid_encode(ids)
      end
    end

    def decode_id(ids)
      if ids.is_a?(Array)
        ids.map { |id| hashid_decode(id) }
      else
        hashid_decode(ids).first
      end
    end

    def find(*ids)
      expects_array = ids.first.is_a?(Array)

      uniq_ids = ids.flatten.compact.uniq
      uniq_ids = uniq_ids.first unless expects_array || uniq_ids.size > 1

      if uniq_ids.is_a?(Array)
        uniq_ids.map do |id|
          id.is_a?(Integer) ? super(id) : super(decode_id(id))
        end
      else
        uniq_ids.is_a?(Integer) ? super(uniq_ids) : super(decode_id(uniq_ids))
      end
    end

    def find_by_hashid(hashid)
      find_by id: decode_id(hashid)
    end

    def find_by_hashid!(hashid)
      find(decode_id(hashid))
    end

    def hashid
      encode_id(id)
    end

    private

    def hashids
      Hashids.new(self.class.name + ENV.fetch('HASHID_TOKEN', ''), 8)
    end

    def hashid_encode(id)
      return nil if id.nil?

      hashids.encode(id)
    end

    def hashid_decode(id)
      hashids.decode(id)
    rescue Hashids::InputError
      id
    end
  end
end
```

Now we just have to add a line to the model that we want to use hashids and it's all done:

```ruby
class User < ApplicationRecord
  include Hashable
end
```

# Slugs: Human readability, URL-friendly 

Slugs represent a human-readable, URL-friendly string derived from a source text, typically used to represent titles, names, or other descriptive elements. 

## Advantages

1. **Readability**: Slugs create human-friendly URLs that are easy to read and understand, improving the overall user experience. Descriptive slugs provide users with valuable context about the content they are accessing.
2. **SEO Optimization**: By including relevant keywords and descriptive elements in the URL, slugs contribute to better search engine visibility and ranking. Search engines often use URLs as part of their ranking algorithms, making SEO-friendly slugs essential for driving organic traffic.
3. **Brand Consistency**: Consistently formatted slugs contribute to brand recognition and consistency across web pages. Users can easily identify content from the URL, reinforcing brand identity and credibility.
4. **Compatibility**: Slugs adhere to web standards and are compatible with various web browsers and platforms. They facilitate easy sharing and linking of content across different channels, including social media and email.

## Disadvantages

1. **Uniqueness**: Ensuring the uniqueness of slugs across a large dataset can be challenging, especially in environments where multiple users can create content simultaneously. Conflicting slugs may lead to URL collisions and accessibility issues.
2. **Character Limitations**: URLs have character limitations, and slugs must adhere to these constraints to ensure compatibility and avoid truncation issues. Long titles or descriptive elements may require truncation or abbreviation, potentially affecting readability.
3. **Text Processing Complexity**: Generating slugs involves processing and sanitizing text inputs to remove special characters, whitespace, and other non-URL-friendly characters. This text processing complexity can introduce performance overhead and require careful handling of edge cases.
4. **Internationalization**: Supporting multilingual content poses challenges for slug generation, as different languages may have unique character sets and conventions. Ensuring consistent and SEO-friendly slugs across languages requires careful consideration and possibly language-specific handling.

## Potentialities

1. **Customization**: Slugs offer flexibility in customization, allowing developers to incorporate branding elements, keywords, and relevant metadata into URLs. Customizable slugs enable tailored SEO strategies and branding initiatives.
2. **SEO Enhancement**: Optimizing slugs for SEO can significantly impact search engine visibility and ranking, driving organic traffic and user engagement. Strategic use of keywords and descriptive elements in slugs improves content discoverability and relevance.
3. **User Engagement**: User-friendly slugs enhance user engagement by providing clear and descriptive URLs that users can easily remember and share. Improved usability and readability contribute to positive user experiences and higher retention rates.
4. **Localization**: Tailoring slugs to different languages and regions facilitates localization efforts, ensuring that content remains accessible and relevant to diverse audiences. Localized slugs contribute to global brand reach and engagement.


## How to do this

You could use a script similar to the one I provided above to override the find method and use the slug as the main parameter or simply change the call in the controller, which would be:

```ruby
@user = User.find(params[:id])
```

To: 

```ruby
@user = User.find_by(slug: params[:id])
```


# Conclusion: My Opinion

So which option should I use? As always the answer is...it depends.

Firstly, sequential ID is rarely a good option unless you want the sequential ID to be part of the functionality, for example you want to show the user that one ID is older than another. At a time when there were few messaging applications and ICQ dominated the world, having an ICQ number with less than 6 digits was a sign of status, that you were "old" on the platform.

As I said in the same session, UUID only makes sense if you want to manipulate decentralized database systems. Keep in mind that a UUID takes up a little more space than an integer, but not that much.

My preferred option is Hash ID, this solution works for 99% of cases. It does not take up extra space in the database, on the other hand it requires a little more encryption/decryption processing. Do I have a user model? Are they invoices? Are they work orders? Throws a Hash ID. Everything's solved.

Finally, if I need the ID to be part of a URL and need to work SEO or have some meaning, the solution is slug. The slug will result in a significant increase in database space and even more creating an index just for it, but in terms of being something more user-friendly it has no equal. Is my user model for sharing with other people (like on Linkedin)? Am I creating IDs for blog posts? There's nothing to think about, it's Slug.

I hope this collection of information about the use of ID has been useful.
