# SmartCoach - AI-Powered Financial Coaching Agent

SmartCoach is an intelligent financial coaching application designed specifically for irregular earners (gig workers, freelancers, students, creators) in India and globally. It provides personalized insights, risk detection, and actionable recommendations to help users build better financial habits.

## 🚀 Features

### Core Functionality
- **Smart Data Ingestion**: Upload CSV/XLSX bank statements, UPI records, and financial data
- **AI-Powered Insights**: Detect spending patterns, income volatility, recurring bills, and anomalies
- **Personalized Coaching**: Get actionable advice, budget suggestions, and timely reminders
- **Budget Management**: Set and track spending limits by category with visual progress indicators
- **Transaction Analysis**: Comprehensive transaction history with smart categorization
- **Risk Detection**: Identify overspending, cash-flow gaps, and unusual charges

### Target Users
- Gig workers and freelancers
- Students and young professionals
- Content creators and influencers
- Small business owners
- Anyone with irregular income patterns

## 🛠️ Tech Stack

- **Frontend**: Next.js 14 (App Router) + TypeScript + Tailwind CSS
- **UI Components**: shadcn/ui component library
- **Authentication**: Clerk
- **Database**: PostgreSQL + Prisma ORM
- **File Processing**: csv-parse, xlsx
- **AI Integration**: OpenAI API (function calling)
- **Email**: Resend
- **Charts**: Recharts (coming soon)
- **Testing**: Vitest + Playwright

## 📋 Prerequisites

- Node.js 18+ 
- PostgreSQL database
- Clerk account for authentication
- OpenAI API key
- Resend account for email

## 🚀 Quick Start

### 1. Clone and Install

```bash
git clone <repository-url>
cd smartcoach
npm install
```

### 2. Environment Setup

Copy `.env.example` to `.env.local` and fill in your credentials:

```bash
cp .env.example .env.local
```

Required environment variables:

```env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/smartcoach"

# Authentication (Clerk)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="pk_test_..."
CLERK_SECRET_KEY="sk_test_..."

# OpenAI API
OPENAI_API_KEY="sk-..."

# Email (Resend)
RESEND_API_KEY="re_..."

# App Configuration
APP_URL="http://localhost:3000"

# Encryption
ENCRYPTION_KEY="your-32-character-encryption-key-here"
```

### 3. Database Setup

```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate dev

# (Optional) Seed with sample data
npx prisma db seed
```

### 4. Start Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## 📁 Project Structure

```
smartcoach/
├── app/                    # Next.js App Router
│   ├── api/               # API routes
│   ├── dashboard/         # Protected dashboard pages
│   ├── sign-in/          # Authentication pages
│   └── sign-up/          # Authentication pages
├── components/            # Reusable UI components
│   ├── ui/               # shadcn/ui components
│   └── ...               # Custom components
├── lib/                   # Utility functions
├── prisma/                # Database schema and migrations
├── public/                # Static assets
└── types/                 # TypeScript type definitions
```

## 🔧 Development

### Available Scripts

```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run type-check   # Run TypeScript compiler
```

### Database Commands

```bash
npx prisma studio           # Open Prisma Studio
npx prisma migrate dev      # Create and apply migration
npx prisma migrate reset    # Reset database
npx prisma db seed         # Seed database
```

### Adding New Components

```bash
npx shadcn@latest add [component-name]
```

## 📊 Sample Data

The app includes sample CSV data for testing:

- **Sample CSV**: `/public/sample-transactions.csv`
- **Format**: date, amount, description, type, counterparty, account, currency
- **Features**: Indian-style dates (DD/MM/YYYY), UPI-style descriptions, mixed income/expense

## 🔐 Security Features

- **Authentication**: Clerk-based user management
- **Data Encryption**: PII encryption at rest
- **Input Validation**: Comprehensive CSV/XLSX parsing with validation
- **Rate Limiting**: API endpoint protection
- **Privacy**: No PII in logs, account deletion support

## 🧪 Testing

### Unit Tests

```bash
npm run test              # Run Vitest tests
npm run test:ui          # Run tests with UI
npm run test:coverage    # Generate coverage report
```

### E2E Tests

```bash
npm run test:e2e         # Run Playwright tests
npm run test:e2e:ui      # Run tests with UI
```

## 🚀 Deployment

### Vercel (Recommended)

1. Connect your GitHub repository to Vercel
2. Set environment variables in Vercel dashboard
3. Deploy automatically on push to main branch

### Manual Deployment

```bash
npm run build
npm run start
```

## 📈 Roadmap

### Phase 1 (Current)
- [x] Basic app structure and authentication
- [x] CSV/XLSX upload and parsing
- [x] Dashboard with sample data
- [x] Budget management UI
- [x] Coach interface

### Phase 2 (Next)
- [ ] OpenAI integration for AI coach
- [ ] Real-time insights and recommendations
- [ ] Email notifications and digests
- [ ] Advanced analytics and charts
- [ ] Mobile app

### Phase 3 (Future)
- [ ] Telegram bot integration
- [ ] Multi-currency support
- [ ] Investment tracking
- [ ] Tax optimization
- [ ] Social features

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ⚠️ Disclaimer

**This is an educational tool, not financial advice.** SmartCoach provides insights and recommendations based on your data, but all financial decisions should be made in consultation with qualified professionals. The app is designed to help you understand your financial patterns and make informed decisions.

## 🆘 Support

- **Issues**: [GitHub Issues](https://github.com/your-repo/smartcoach/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-repo/smartcoach/discussions)
- **Documentation**: [Wiki](https://github.com/your-repo/smartcoach/wiki)

## 🙏 Acknowledgments

- Built with [Next.js](https://nextjs.org/)
- UI components from [shadcn/ui](https://ui.shadcn.com/)
- Authentication by [Clerk](https://clerk.com/)
- Database powered by [Prisma](https://www.prisma.io/)
- AI capabilities from [OpenAI](https://openai.com/)

---

**Made with ❤️ for irregular earners everywhere**
