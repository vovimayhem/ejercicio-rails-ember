# Stage 1: Runtime
# Start off from the ruby official image, alpine version:
FROM ruby:2.4.3-alpine AS runtime

# Set the home to the code path, and
# add the code executables to $PATH:
ENV HOME=/usr/src PATH=/usr/src/bin:$PATH

WORKDIR /usr/src

# Add the runtime packages (needed for the app to run):
RUN apk add --no-cache \
  ca-certificates \
  less \
  libpq \
  openssl \
  tzdata

# Stage 2: Development
# Pick off from runtime image:
FROM runtime AS development

# Add the development packages (compilers, dev headers, etc):
RUN apk add --no-cache \
  build-base \
  chromium \
  chromium-chromedriver \
  git \
  nodejs \
  postgresql-dev

# Copy the project's Gemfile
COPY Gemfile* /usr/src/

# Download & Install the project's ruby libraries:
RUN bundle install --jobs=4 --retry=3

# Set the default command for development images:
CMD ["rails", "server", "-b", "0.0.0.0", "-p", "3000"]

# Stage 3: Builder
# Pick off from development image:
FROM development AS builder

# Copy the rest of the code:
COPY . /usr/src

# Precompile assets:
RUN DATABASE_URL=postgres://postgres@example.com:5432/fakedb \
 SECRET_KEY_BASE=10167c7f7654ed02b3557b05b88ece \
 RAILS_ENV=production \
 rails assets:precompile

# Cleanup the ruby libraries not used in a productive context:
RUN bundle config without development:test \
 && bundle clean

# Stage 4: Deployable
# Pick off from runtime image:
FROM runtime AS deployable

# Copy the ruby libraries remaining after the cleanup:
COPY --from=builder /usr/local/bundle /usr/local/bundle

# Copy the code including the compiled assets:
COPY --from=builder /usr/src /usr/src

# Set the environment name variables & default PORT:
ENV RAILS_ENV=production RACK_ENV=production PORT=3000

# Set the default command for deployable images:
CMD ["puma"]
