# 🚀 Frontend Deployment Guide - Vercel

## 📋 Pre-Deployment Checklist

### ✅ Configuration Ready
- **Environment Variables**: Updated to production backend
- **Vercel Config**: `vercel.json` configured for SPA routing
- **Build Script**: `npm run build` ready
- **Dependencies**: All packages installed

### 🔧 Current Configuration
```env
REACT_APP_API_URL=https://shksabbir7.pythonanywhere.com/api
REACT_APP_MEDIA_URL=https://shksabbir7.pythonanywhere.com
GENERATE_SOURCEMAP=false
```

## 🌐 Deploy to Vercel

### Method 1: Vercel CLI (Recommended)

1. **Install Vercel CLI**
```bash
npm install -g vercel
```

2. **Login to Vercel**
```bash
vercel login
```

3. **Deploy from Frontend Directory**
```bash
cd frontend
vercel --prod
```

4. **Follow Prompts**
- Set up and deploy? **Y**
- Which scope? **Select your account**
- Link to existing project? **N** (first time)
- Project name? **tastestack-frontend** (or your choice)
- Directory? **./** (current directory)

### Method 2: GitHub Integration

1. **Push to GitHub** (already done)
2. **Go to [vercel.com](https://vercel.com)**
3. **Import Project**
   - Connect GitHub account
   - Select `TasteStack-A-Recipe-Sharing-Platform` repository
   - Set root directory to `frontend`
4. **Configure Build Settings**
   - Framework Preset: **Create React App**
   - Build Command: `npm run build`
   - Output Directory: `build`
   - Install Command: `npm install`

### Method 3: Drag & Drop

1. **Build Locally**
```bash
cd frontend
npm run build
```

2. **Upload to Vercel**
   - Go to [vercel.com/new](https://vercel.com/new)
   - Drag the `build` folder to upload

## ⚙️ Environment Variables Setup

In Vercel dashboard, add these environment variables:

```
REACT_APP_API_URL = https://shksabbir7.pythonanywhere.com/api
REACT_APP_MEDIA_URL = https://shksabbir7.pythonanywhere.com
GENERATE_SOURCEMAP = false
```

## 🔧 Vercel Configuration

Your `vercel.json` is already configured:
```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "build"
      }
    }
  ],
  "routes": [
    {
      "src": "/static/(.*)",
      "dest": "/static/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

## 🧪 Post-Deployment Testing

After deployment, test these features:

### 1. **Basic Functionality**
- [ ] Homepage loads
- [ ] Navigation works
- [ ] API connection established

### 2. **Authentication**
- [ ] Registration form works
- [ ] Login with backend API
- [ ] JWT token handling

### 3. **Recipe Features**
- [ ] Recipe list displays
- [ ] Recipe details load
- [ ] Image loading from backend

### 4. **API Integration**
- [ ] All API calls work
- [ ] CORS configured properly
- [ ] Error handling works

## 🔍 Troubleshooting

### Common Issues:

**1. API Connection Failed**
```
Solution: Check CORS settings in Django backend
Verify: CORS_ALLOW_ALL_ORIGINS = True in production
```

**2. Images Not Loading**
```
Solution: Check REACT_APP_MEDIA_URL points to backend
Verify: https://shksabbir7.pythonanywhere.com/media/ accessible
```

**3. Routing Issues**
```
Solution: Ensure vercel.json routes configuration
All routes should redirect to /index.html for SPA
```

**4. Build Errors**
```bash
# Check build locally first
npm run build

# Fix any warnings or errors
# Then redeploy
```

## 📊 Expected Results

After successful deployment:

- **Frontend URL**: `https://your-app.vercel.app`
- **Status**: ✅ Live and connected to backend
- **Features**: Full recipe platform functionality
- **Performance**: Fast loading with Vercel CDN

## 🔄 Continuous Deployment

Vercel automatically redeploys when you push to GitHub:

1. **Make changes** to frontend code
2. **Commit and push** to GitHub
3. **Vercel automatically builds** and deploys
4. **New version live** in ~2 minutes

## 🎯 Next Steps After Deployment

1. **Test all functionality**
2. **Update README** with live frontend URL
3. **Share live demo** links
4. **Monitor performance** in Vercel dashboard
5. **Set up custom domain** (optional)

---

**Ready to deploy?** Run `vercel --prod` from the frontend directory!