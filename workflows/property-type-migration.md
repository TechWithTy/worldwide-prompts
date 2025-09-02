---
description: Migrate property types and update related components
---
# Property Type Migration Workflow

This workflow outlines the steps to update property types and related components to use the new `Property` type union (`RealtorProperty | RentCastProperty`).

## 1. Update Type Definitions

1. **Review and Update `property.ts`**
   - Ensure all property-related types are properly defined
   - Verify type guards (`isRealtorProperty`, etc.) are working correctly
   - Add any missing utility types

2. **Update Property Creation Utilities**
   - Review `createRealtorProperty` and `createRentCastProperty`
   - Ensure they handle all required fields with proper defaults

## 2. Update Components

### 2.1 Property Page Components

1. **Property Overview Card**
   - [ ] Update props to accept new `Property` type
   - [ ] Handle both property types with type guards
   - [ ] Add proper fallbacks for missing data
   - [ ] Update any child components to use new property structure

2. **Property List**
   - [ ] Update `PropertyListProps` to use new `Property` type
   - [ ] Update property card rendering to handle both property types
   - [ ] Add proper type checking for property-specific fields

3. **Property Card**
   - [ ] Update to work with new property structure
   - [ ] Add type guards for property-specific fields
   - [ ] Ensure all data access is type-safe

## 3. Update Data Fetching

1. **API Routes**
   - [ ] Update API endpoints to return new property types
   - [ ] Add proper type transformations if needed
   - [ ] Ensure backward compatibility if needed

2. **Data Transformation**
   - [ ] Add utility functions to transform between old and new types
   - [ ] Update any data normalization logic

## 4. Testing

1. **Unit Tests**
   - [ ] Update existing tests for new types
   - [ ] Add tests for type guards
   - [ ] Test edge cases with partial data

2. **Integration Tests**
   - [ ] Test property page rendering with both property types
   - [ ] Verify all data is displayed correctly
   - [ ] Test error handling for missing data

## 5. Migration Steps

1. **Backward Compatibility**
   - [ ] Create migration path for existing data
   - [ ] Add deprecation warnings for old types
   - [ ] Plan for removal of deprecated types

2. **Deployment**
   - [ ] Deploy API changes first
   - [ ] Deploy frontend changes
   - [ ] Monitor for any type-related errors

## 6. Documentation

1. **Type Documentation**
   - [ ] Document new property type structure
   - [ ] Add examples for common use cases
   - [ ] Document migration path for consumers

2. **Component Documentation**
   - [ ] Update component documentation
   - [ ] Add examples for both property types
   - [ ] Document any breaking changes

## 7. Verification

1. **Smoke Testing**
   - [ ] Verify all property pages load correctly
   - [ ] Check property listings and search
   - [ ] Test property creation/editing

2. **Performance**
   - [ ] Check for any performance regressions
   - [ ] Optimize any slow type guards or transformations

## Rollback Plan

1. **If Issues Arise**
   - Revert to previous type definitions
   - Temporarily restore any removed code
   - Add more comprehensive tests

2. **Post-Mortem**
   - Document any issues encountered
   - Update the workflow with lessons learned
   - Plan for future type migrations
