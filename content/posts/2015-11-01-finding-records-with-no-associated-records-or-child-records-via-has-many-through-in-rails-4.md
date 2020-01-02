---
layout: post
title: Finding records with no associated records (where child association does not exist) via has_many :through in Rails 4
date: '2015-11-01'
excerpt: "A pretty simple way to find records that have no associated records"
tags: [ruby, activerecord, rails]
modified: 2015-11-01
date: 2015-10-21
comments: true
---

Suppose you have the following Activerecord associations established:

{{< highlight ruby >}}
Class Student 
  has_many :class_sessions
end

Class ClassSession
  belongs_to :student
end
{{< /highlight >}}

And you want to find out which students have no class sessions.  Often I've seen some interesting solutions like:

{{< highlight ruby >}}
Student.all - Student.find(ClassSession.pluck(:student_id))
{{< /highlight >}}

But this can get really bad if you ever have any reasonable amount of data to deal with.  You can get what you want with a simple query:

{{< highlight ruby >}}
Student.includes(:class_sessions).where(class_sessions: {id: nil})
{{< /highlight >}}

When you use "includes" activerecord will use LEFT OUTER JOIN to include whatever associations you are asking for.  In the result set, if associated records exist in the related tables, their values will will be included.  If the associated record(s) don't exist, the values for columns on the included association will be nil.

Which means if you check those associated columns for nil, you are essentially checking that the association is absent for that row.  Hence,

{{< highlight ruby >}}
Student.includes(:class_sessions).where(class_sessions: {id: nil})
{{< /highlight >}}

Will, by checking for the absence of the class_sessions.id column, give you all the students without an associated class session.

What's nice is that this applies to indirect associations via has_many through.  Consider this arrangement where a student has class sessions through a join table called enrollments:

{{< highlight ruby >}}
Class Student 
  has_many :class_sessions, through: :enrollments
  has_many :enrollments
end

Class Enrollment
  belongs_to :student
  belongs_to :class_session
end

Class ClassSession
  has_many :enrollments
  has_many :students, through: :enrollments
end
{{< /highlight >}}

How would you check to find students without any class sessions here?  Well, it's the same:

{{< highlight ruby >}}
Student.includes(:class_sessions).where(class_sessions: {id: nil})
{{< /highlight >}}

Or if you wanted to be needlessly obtuse:

{{< highlight ruby >}}
Student.includes(:enrollments).where(enrollments: {student_id: nil})
{{< /highlight >}}

It's probably better to check for nil on the actual associated record and not the join table, just so it is clear what you are checking for.

Ok let's get crazy--what if you want to check on a really indirect association.  So building off what we had before lets add two more classes:

{{< highlight ruby >}}
Class Student 
  has_many :class_sessions, through: :enrollments
  has_many :enrollments
  has_many :teachers, through: :class_sessions
end

Class Enrollment
  belongs_to :student
  belongs_to :class_session
end

Class ClassSession
  has_many :enrollments
  has_many :students, through: :enrollments
end

Class Timeslot
  belongs_to :teacher
  belongs_to :class_session
end

Class Teacher
  has_many :timeslots
  has_many :class_sessions, through: :timeslots
  has_many :students, through: :class_sessions
end
{{< /highlight >}}

Er..this example is getting a little convoluted but it could happen.  Can you form a simple query to find students with no teachers?  Sure!

{{< highlight ruby >}}
Student.includes(:teachers).where(teachers: {id: nil})
{{< /highlight >}}

The SQL that Activerecord generates is pretty gnarly but the Ruby/Rails is simple if not elegant.

{{< highlight ruby >}}
SELECT "students"."id" AS t0_r0, "students"."name" AS t0_r1, "students"."created_at" AS t0_r2, "students"."updated_at" AS t0_r3, "teachers"."id" AS t1_r0, "teachers"."name" AS t1_r1, "teachers"."created_at" AS t1_r2, "teachers"."updated_at" AS t1_r3 FROM "students" LEFT OUTER JOIN "enrollments" ON "enrollments"."student_id" = "students"."id" LEFT OUTER JOIN "class_sessions" ON "class_sessions"."id" = "enrollments"."class_session_id" LEFT OUTER JOIN "timeslots" ON "timeslots"."class_session_id" = "class_sessions"."id" LEFT OUTER JOIN "teachers" ON "teachers"."id" = "timeslots"."teacher_id" WHERE "teachers"."id" IS NULL
{{< /highlight >}}

Yep so that's one way to look for unassociated primary records.  If anyone has a better way or thinks I've said something stupid, let me know!  I'm always ready to level up.

