FROM ruby:2 AS build

RUN gem install bundler

RUN mkdir -p /workspace/__site

WORKDIR /workspace

RUN cd /workspace
RUN git clone https://github.com/edgarkenji/edgarkenji.github.io.git ./__site

RUN cp -R /workspace/__site/* /workspace

RUN bundle install

ENV JEKYLL_ENV=production
RUN bundle exec jekyll build

FROM nginx:1

WORKDIR /app
COPY --from=build /workspace/_app /app
COPY --from=build /workspace/_site /app/site

CMD ["/app/start.sh"]
