FROM ruby:2 AS build

RUN gem install bundler

#Install git
RUN apt-get update \        
	apt-get install -y git

WORKDIR /workspace

RUN git clone https://github.com/edgarkenji/edgarkenji.github.io.git

COPY Gemfile* /workspace/
RUN bundle install

COPY . /workspace
ENV JEKYLL_ENV=production
RUN bundle exec jekyll build


FROM nginx:1

WORKDIR /app
COPY _app /app
COPY --from=build /workspace/_site /app/site

CMD ["/app/start.sh"]
