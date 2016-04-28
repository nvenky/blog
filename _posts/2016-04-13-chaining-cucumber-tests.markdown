---
layout: post
title:  "Triggering a cucumber test from another test"
subtitle: "Chaining cucumber tests"
date:   2016-04-13 09:00:01
categories: [cucumber]
---

I was working on a legacy web application that has limited unit test and dependency on multiple applications.  To reduce the manual testing effort, we had to write end-to-end UI tests with help of Cucumber, Capybara.

We had to chain few tests (generally BAD PRACTICE) as there was no way to create the seed data in every end-to-end test. We had to reuse the data from other scenarios. Rather than going into further details, here is an example where you can trigger another cucumber tag from a step:

## child_process.feature

{% highlight javascript %}
Feature: As a Cucumber user
  I want to run other feature from a step

  @success_child_process
  Scenario: run job
    Given no exception is raised

  @failure_child_process
  Scenario: run job
    Given exception is raised

  Scenario: Child
    Given I trigger @success_child_process job
    And no exception is raised
    And I trigger @failure_child_process job
{% endhighlight %}

## child_process_steps.rb

{% highlight ruby %}
  Given /^no exception is raised$/ do
    # do nothing here
  end

  Given /^exception is raised$/ do
    raise 'Exception is raised'
  end

  Given /^I trigger (@.*) job$/ do |tag|
    CucumberChildProcess.run_tag tag
  end
{% endhighlight %}


## cucumber_child_process.rb

We use the **childprocess** gem to run the child tag as a seperate process

{% highlight ruby %}
require 'childprocess'

module CucumberChildProcess

  def self.run_tag(tag)
    logger.info "Triggering the child process for tag '#{tag}'"
    process = ChildProcess.build(find_executable('bundle'), 'exec', 'cucumber', '-t', tag)
    process.io.inherit!
    process.start
    logger.info "********* CHILD PROCESS LOGS for tag '#{tag}' - START **********"
    process.wait
    logger.info "********* CHILD PROCESS LOGS for tag '#{tag}' - END **********"
    raise "Child process failed while running the tag '#{tag}'. Check the log for more information. Exit code: #{process.exit_code}" unless process.exit_code == 0
    logger.info "Successfully executed the tag '#{tag}' as Child process"
  end

  def self.find_executable(cmd)
    ENV['PATH'].split(File::PATH_SEPARATOR).each do |path|
      next if path.empty?
      path = File.join(path, cmd)
      ['.exe', '.bat'].each do |ext|
        return path + ext if File.exists?(path + ext)
      end
    end
    raise "Unable to find command '#{cmd}' in the PATH"
  end
end
{% endhighlight %}
