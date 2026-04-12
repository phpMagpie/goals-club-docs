# Seed Activities

Activities to seed after stack deployment. These are real completions from Paul & Lynsey that should be added to the database.

---

## Paul Gardner - Summit All 214 Wainwrights

| Peak | Date | Notes |
|------|------|-------|
| Catbells | 12/04/2023 | Really popular, wore brand new Van boots, bad mistake. Enjoyed the views and the dachshunds, did not enjoy the foot pain. |

---

## Lynsey Hall - Summit All 214 Wainwrights

| Peak | Date | Notes |
|------|------|-------|

---

## How to Seed

After running `make deploy` in goals-club-data, these activities can be seeded by:

1. **Manual seeder** - Add to `packages/database/seeders/` as a new seeder file
2. **API calls** - Use the `logGoalActivity` mutation via the GraphQL API
3. **Direct SQL** - Insert directly via AWS RDS Data API

### Example SQL for seeding:

```sql
-- First, get the user_goal_id for Paul's Wainwrights goal
SELECT ug.id as user_goal_id, gi.id as goal_item_id, gi.name
FROM user_goals ug
JOIN users u ON ug.user_id = u.id
JOIN goals g ON ug.goal_id = g.id
JOIN goal_items gi ON gi.goal_id = g.id
WHERE u.email = 'paul@thegoalsclub.co.uk'
  AND g.title LIKE '%Wainwright%'
  AND gi.name = 'Catbells';

-- Then insert the activity
INSERT INTO user_goal_activities (
  id, user_goal_id, activity_type, goal_item_id, 
  activity_date, is_completion, notes, created_at
) VALUES (
  UUID(), '<user_goal_id>', 'ITEM_COMPLETION', '<goal_item_id>',
  '2023-04-12', 1, 'Really popular, wore brand new Van boots, bad mistake. Enjoyed the views and the dachshunds, did not enjoy the foot pain.',
  NOW()
);
```

---

## Future: Automated Seeding

Consider creating a dedicated seeder file that reads from this document or a JSON file to automatically seed these activities during deployment.

